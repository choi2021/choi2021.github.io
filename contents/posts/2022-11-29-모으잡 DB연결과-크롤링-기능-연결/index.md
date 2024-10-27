---
title: "모으잡-DB연결과 크롤링 기능 연결"
date: 2022-11-29
slug: 모으잡-DB연결과-크롤링-기능-연결
tags: [사이드프로젝트, 모으잡]
series: 모으잡
---

## 📁DB연결

개인 별 채용공고들의 정보를 저장하기 위해서 Firebase의 realtime database를 추가해 연결했다. firebaseApp은 이미 인증/인가 기능을 추가하면서 만들어 두었기 때문에 DBService의 interface를 이용한 class를 추가해 구현했고, 해당 api를 이용할 때는 react-query를 이용했다.

DB는 이전 채용공고들을 받아오는 getJobs, 새로운 채용공고를 추가하는 addJob, 채용공고를 삭제하는 deleteJob, 채용공고를 변경할 수 있는 updateJob 4가지를 interface로 만들었다.

원래 updateJob은 생각하지 않았지만, 이후 해당 공고에 체크 기능을 추가하고 체크된 비율을 공고에 담아줘야 하기 때문에 추가하게 된 기능이다. updateJob과 addJob을 합쳐서 setJob으로 처리해도 되지 않을까라는 생각을 했지만 우선은 나눠서 처리했다.

```typescript
export interface DBService {
  addJob: (job: ModifiedJobType) => Promise<void>
  getJobs: () => Promise<ModifiedJobsType>
  removeJob: (job: ModifiedJobType) => Promise<void>
  updateJob: (job: ModifiedJobType) => Promise<void>
}

export class DBServiceImpl implements DBService {
  db: Database
  constructor(private app: FirebaseApp) {
    this.db = getDatabase(this.app)
  }

  addJob(job: ModifiedJobType) {
    const userId = localStorage.getItem(UserId)
    return set(ref(this.db, `users/${userId}/jobs/${job.id}`), job)
  }

  updateJob(job: ModifiedJobType) {
    const userId = localStorage.getItem(UserId)
    return set(ref(this.db, `users/${userId}/jobs/${job.id}`), job)
  }

  async getJobs(): Promise<ModifiedJobsType> {
    const userId = localStorage.getItem(UserId)
    const dbRef = ref(this.db)
    return get(child(dbRef, `users/${userId}/jobs`))
      .then(snapshot => {
        if (snapshot.exists()) {
          return snapshot.val()
        } else {
          return []
        }
      })
      .catch(error => {
        console.error(error)
      })
  }

  removeJob(job: ModifiedJobType) {
    const userId = localStorage.getItem(UserId)
    return remove(ref(this.db, `users/${userId}/jobs/${job.id}`))
  }
}
```

### 1. getJobs

getJobs를 사용하는 곳은 JobList 컴포넌트로 메인페이지와 상세페이지에서 채용공고들을 보여주기 위해 필요하다. 상세 페이지의 경우 보여주고 있는 페이지는 제외해 주어야 하기 때문에 react-query의 select를 이용해서 filtering을 한 data를 전달하게 로직을 구성했다. react-query를 이용해 별도의 hook을 만들지 않고 loading 상태를 관리할 수 있어서 더 편하게 작업할 수 있었다.

```tsx
export default function JobList() {
  const { query } = useRouter()
  const { id } = query
  const dbService = useDBService()
  const { data: jobs, isLoading } = useQuery(
    ["jobs"],
    () => dbService.getJobs(),
    {
      select: (data: ModifiedJobsType) => {
        return Object.values(data).filter(item => item.id !== id)
      },
    }
  )

  if (isLoading) {
    return <div>채용공고를 불러오는 중입니다...</div>
  }
  return (
    <Wrapper>
      {jobs && jobs.map(job => <JobItem key={job.id} job={job} />)}
    </Wrapper>
  )
}
```

### 2. removeJob

deleteJob은 JobList로 불러온 공고들의 삭제버튼에 연결되어야 할 기능으로 react-query의 useMutation을 이용해 연결했고, UI상의 변화와 불러온 서버 데이터를 동기화 시키기 위해서 onSuccess로 invalidtateQueries로 새로 DB에서 채용공고들을 받아오게 했다.

```tsx
export default function JobItem({ job }: { job: ModifiedJobType }) {
  const { name, platform, img, checkPercentage } = job
  const queryClient = useQueryClient()
  const dbService = useDBService()
  const { mutate } = useMutation(
    async (job: ModifiedJobType) => {
      return dbService.removeJob(job)
    },
    {
      onSuccess: () => {
        queryClient.invalidateQueries(["jobs"])
      },
      onError: error => {
        if (error instanceof AxiosError) {
          const { response } = error
          if (response) {
            console.log(response)
          }
        }
      },
    }
  )
  const handleDelete = () => {
    mutate(job)
  }
  const over50Percent = checkPercentage >= 0.5

  return (
    <Wrapper>
      {over50Percent && <Badge>50% 이상</Badge>}
      <DeleteBtn onClick={handleDelete}>
        <MdRemove />
      </DeleteBtn>
      <Link href={`/jobs/${job.id}`}>
        <Img src={img} alt="job" width="200" height="180" priority />
        <Box>
          <h1>{name}</h1>
          <h3>{platform}</h3>
        </Box>
      </Link>
    </Wrapper>
  )
}
```

### 3. addJob

addJob은 새로운 채용공고를 받아올 때 연결이 필요한 부분으로, jobForm 컴포넌트에서 서버로 url을 전달하고 크롤링한 데이터를 채용공고로 가져왔을 때 채용공고를 추가하기 위한 기능이다. useMutation을 이용해 서버로 api호출을 보낸 결과를 받아와 추가했다. 이때 체크 기능을 위해서 데이터의 format이 필요했고 addCheckToJob을 이용해 string[]이었던 preferential과 qualification을 체크된 여부를 담은 오브젝트의 배열로 변환하고, 체크된 비율을 추가해주었다.

```tsx
//setChecks.ts

const addCheckToJob = (job: JobType): ModifiedJobType => {
  const preferential = job.preferential.map((item) => ({
    text: item,
    checked: false,
  }));
  const qualification = job.qualification.map((item) => ({
    text: item,
    checked: false,
  }));
  const checkPercentage = 0;
  return { ...job, preferential, qualification, checkPercentage };
};

//JobForm.tsx

export default function JobForm() {
  const [url, setUrl] = useState('');
  const dbService = useDBService();

  const queryClient = useQueryClient();
  const { mutate, isLoading } = useMutation(
    async (url: string) => {
      const { data } = await axios.post(
        `${process.env.NEXT_PUBLIC_HOST}/api/job`,
        {
          url,
        }
      );
      const job = addCheckToJob(data);
      dbService.addJob(job);
    },
    {
      onSuccess: () => {
        queryClient.invalidateQueries(['jobs']);
      },
      onError: (error) => {
        if (error instanceof AxiosError) {
          const { response } = error;
          if (response) {
            setUrl('');
            setMessage(response?.data.message);
          }
        }
      },
    }
  );
	...
  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    if (!url) {
      return;
    }
    mutate(url);
  };

  return (
   	...
  );
}

```

### 4. updateJob

updateJob은 상세 페이지에서 해당 공고의 check여부를 바꿔주기 위해 useMutaion을 이용해 연결되었다. updateJob에서 해당 job을 통으로 넘겨줘야하기 때문에 캐쉬된 채용공고들을 받아 id로 찾는 로직이 필요했다. 정리하면서 아쉬웠던 점은 handleChange의 코드를 보면 자격 조건의 DescriptionItem인지 우대 사항의 DescriptionItem인지를 알려 주는 kind를 전달 받기 때문에, 로직을 kind에 따라 수정할 수 있을 것 같은데 동일한 로직을 두번 사용한 부분이다. 이후에 리팩토링이 필요해 보였다.

```tsx
export default function DescriptionItem({
  text,
  isMainJob,
  checked,
  kind,
}: DescriptionItemProps) {
  const queryClient = useQueryClient();
  const dbService = useDBService();
  const { query } = useRouter();
  const { id } = query;
  const jobId = typeof id === 'string' ? id : id?.join() || '';
  const { data: job } = useQuery(
    ['jobs'],
    () => {
      return dbService.getJobs();
    },
    {
      select: (data: ModifiedJobsType) => {
        return data[jobId];
      },
    }
  );
  const { mutate } = useMutation(
    async (job: ModifiedJobType) => {
      return dbService.updateJob(job);
    },
    {
      onSuccess: () => {
        queryClient.invalidateQueries(['jobs']);
      },
      onError: (error) => {
        if (error instanceof AxiosError) {
          const { response } = error;
          if (response) {
            console.log(response);
          }
        }
      },
    }
  );
  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const { name } = e.currentTarget;
    let modifiedJob;
    if (job) {
      if (kind === Kinds.qualification) {
        const qualification = [...job?.qualification].map((item) => {
          if (item.text === name) {
            return { ...item, checked: !item.checked };
          }
          return item;
        });
        modifiedJob = { ...job, qualification };
      } else {
        const preferential = [...job?.preferential].map((item) => {
          if (item.text === name) {
            return { ...item, checked: !item.checked };
          }
          return item;
        });
        modifiedJob = { ...job, preferential };
      }
      mutate(calculateChecks(modifiedJob));
    }
  };
  return (
  	...
  );
}

```

이렇게 4개의 기능을 각각의 컴포넌트에 연결해 제대로 작동하는 것을 확인했지만, useMutation이나 useQuery를 반복해서 사용했기 때문에 hook으로 따로 분리하는 리팩토링이 필요해 보였다.

## 🗃 서버 로직 수정

기존에는 서버를 express을 이용해 따로 분리해서 사용했지만, firebase를 이용한 인증/인가 기능과 데이터베이스를 처리하면서 프론트 단에서 하는 일이 많아져 굳이 서버를 분리해서 사용해야 할까란 고민이 되었고, next js로 migration을 진행하면서 next js 자체가 가지는 장점인 api routes기능을 이용해서 처리해보자는 결론을 내렸다.

기존 서버 로직을 pages/api 내부로 가져왔는데, express를 사용하지 않고 node.js와 next내부 기능을 이용해서 서버 로직을 구성했다.

```typescript
import { NextApiRequest, NextApiResponse } from "next"
import Crawler from "./service/CrawlerService"

const crawler = new Crawler()
const POST = "POST"

const JobAPI = async (req: NextApiRequest, res: NextApiResponse) => {
  if (req.method === POST) {
    const url = req.body.url
    try {
      const job = await crawler.createJob(url)
      res.status(201).json(job)
    } catch (error) {
      const Err = error as { message: string }
      res.status(400).json({ message: Err?.message })
    }
  } else {
    res.status(404).json({ message: "잘못된 접근입니다" })
  }
}

export default JobAPI
```

api에서 크롤링 프로그램을 진행하기 위해서는 내부 파일로 crawlerService를 만들어야 해서 간단히 추가했다. 기존 원티드 채용공고를 크롤링을 하면서 가진 문제점은 '•'을 기준으로 자르다 보니 "-"로 정리해놓은 채용공고가 처리가 안되는 문제점이 있었다. 그리고 보다 세부적인 분석을 위해서 JQuery의 selector를 어떻게 처리하는지를 조금 더 공부해 원티드 채용공고에서 필요한 정보가 특정 class Name의 section 안에 다 있다는 것을 확인했다.

개선한 방법은 다음과 같다.

1. className으로 좀 더 세부적인 시작 포인트를 잡는다.
2. 보통 하나의 채용공고 내부 설명이 '•' 나 "-"로 구분되기 때문에 포함여부를 확인하고 기준으로 잡는다.
3. 간혹 우대 사항 아래에 여러 부가 적인 설명이 있는 경우가 있어, html 정보중 `<br/><br/>`를 기준으로 endpoint로 자른 다음에 해당 정보를 가져온다.

```typescript
import cheerio from 'cheerio';
import { JobType } from '../../../types/Jobtype';
import Chrome from 'chrome-aws-lambda';
import puppeteer from 'puppeteer';

type TargetType = {
  [key: number]: 'mainWork' | 'qualification' | 'preferential';
};
const MAINWORK = '주요업무';
const QUALIFICATION = '자격요건';
const PREFERENTIAL = '우대사항';
const DOT_BASE = '•';
const HYPHEN_BASE = '-';

const DotRegex = /(?<=• )(.*?)(?=<br/>)/gm;

const WANTED_URL = 'https://www.wanted.co.kr/wd';

export default class Crawler {
  constructor(private wantedURL: string = WANTED_URL) {}

  checkUrl(url: string) {
    return url.startsWith(this.wantedURL);
  }

  async createJob(url: string) {
    if (!this.checkUrl(url)) {
      throw new Error('url 에러');
    }
    const browser = await puppeteer.launch({
      args: [...Chrome.args, '--hide-scrollbars', '--disable-web-security'],
      defaultViewport: Chrome.defaultViewport,
      executablePath: await Chrome.executablePath,
      headless: true,
      ignoreHTTPSErrors: true,
    });
    const page = await browser.newPage();
    await page.goto(url);

    const content = await page.content();
    const $ = cheerio.load(content);
    const imgLists = $('img');
    const name = $('h6');
    const result: JobType = {
      name: $(name[0]).text(),
      platform: 'wanted',
      id: Date.now().toString(),
      mainWork: [],
      qualification: [],
      preferential: [],
      url,
      img: '',
    };

    imgLists.each((idx, node) => {
      if (idx === 1) {
        result.img = $(node).attr('src') || '';
      }
    });

    const titleList = $('.JobDescription_JobDescription__VWfcb > h6');
    if (titleList.length === 0) {
      throw new Error('Content 에러');
    }
    const contentList = $(
      '.JobDescription_JobDescription__VWfcb > h6+p > span'
    );

    const target: TargetType = {};

    titleList.each((idx, node) => {
      const text = $(node).text();
      switch (text) {
        case MAINWORK:
          target[idx] = 'mainWork';
          break;
        case QUALIFICATION:
          target[idx] = 'qualification';
          break;
        case PREFERENTIAL:
          target[idx] = 'preferential';
          break;
        default:
      }
    });

    contentList.each((idx, node) => {
      if (idx in target) {
        const html = $(node).html()!;
        const isDot = !!html.match(DotRegex);
        const base = isDot ? DOT_BASE : HYPHEN_BASE;
        const endPoint = html.search('<br/><br/>');
        const data = html
          .slice(0, endPoint)
          .split(base)
          .join('')
          .split('<br/>')
          .filter((item) => !!item);
        result[target[idx]] = data;
      }
    });

    await browser.close();
    return result;
  }
}
```

수정을 했지만 여전히 문제는 아직 남아 있었다. 공고 중에서 '•'와 "-"를 동시에 사용하는 공고가 있는 것이다.... (둘 중 하나만 써주세요 ㅜㅜ) 그리고 기존의 틀을 유지 않고 주요업무/ 자격조건/ 우대사항으로 나누지 않고 비교적 자유롭게 작성된 공고도 있었다. 지금에서는 더 처리할 수 없을 것 같아 우선은 이슈로 남기고 점점 세부적인 처리를 추가해나갈 예정이다.

## 마치며

2일 동안 나름의 고민과 사투를 하면서 프로토타입을 만들었다. 아직 해야 할 일들이 너무 많지만, 이번 한번 만들고 끝낼 프로젝트가 아니라 계속해서 수정하고 고도화 시키고 확장시킬 아이디어들이 떠오르고 있다는 점이 설렌다. 배포 후 스터디에서 피드백으로 부족한 점들, 칭찬도 들으니 더 기쁜 맘으로 계속 발전시킬 것 같다. 정말 내 목표대로 나와 같은 취준생 분들이 사용할 수 있는 서비스가 될 수 있기를 기대해본다.
