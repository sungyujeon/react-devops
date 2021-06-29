# React 프로젝트 기반 DevOps 개발환경 실습

[facebook docs 참조](https://create-react-app.dev/docs/deployment/)

- [x] React 프로젝트 생성 및 로컬 실행 확인

- [x] GitHub에 코드 Push 및 Pages에 수동 배포

- [ ] Github Actions workflow 배포 자동화

  \>> github actions 자동화 시 기존 방법을 사용하면 404 File not found



> 목표
>
> - React 프로젝트 생성 및 로컬 실행 확인
> - GitHub에 코드 Push 및 Pages에 수동 배포
> - GitHub Actions workflow로 배포 자동화
> - 코드 수정 및 테스트 실패로 인한 자동 배포 실패 확인



## React 프로젝트 생성 및 로컬 실행 확인

##### Yarn으로 설치하기

> yarn vs npm
>
> - Package 병렬 설치
>   - npm은 여러 package를 설치할 때 각각의 package가 설치된 후 다음이 설치되지만, yarn은 병렬로 처리하여 performance와 speed 증가
> - 자동 Lock file 생성
>   - npm, yarn 모두 package.json에 버전을 명시하고 의존성을 추적 관리함
>   - yarn은 자동으로 yarn.lock 파일 생성하지만, npm은 `npm shrinkwrap` 커맨드로 생성
> - 보안
>   - npm은 다른 package를 즉시 포함시킬 수 있는 코드를 자동으로 시작하여 보안에 취약하지만, yarn은 yarn.lock 또는 package.json에 있는 파일만 설치해 더 안전하다고 여겨짐



##### React 프로젝트 생성

- React 프로젝트 생성

  ```bash
  $ yarn create react-app react-devops
  ```



## GitHub에 코드 Push 및 Pages에 수동 배포

- Create a GitHub repo

- Push a code to GitHub repo

- Github pages로 배포하기 위한 라이브러리 추가 및 배포에 필요한 정보 등록

  - The `predeploy` script will run automatically before `deploy` is run.

  ```bash
  $ yarn add gh-pages -D
  ```

  ```json
  // /pacakge.json
  // predeploy, deploy, clean 추가
  {
    "scripts": {
      "predeploy": "npm run build",
  		"deploy": "gh-pages -d build",
      "start": "react-scripts start",
      "build": "react-scripts build",
      "test": "react-scripts test",
      "eject": "react-scripts eject"
    },
  }
  ```

- 배포용 homepage 필드 추가

  - Create React App uses the `homepage` field to determine the root URL in the built HTML file.

  ```json
  // package.json
  {
   "homepage": "http://dianaleee.github.io/react-deploy-test"
  }
  ```

- build

  - 빌드 시 정적 파일을 원격저장소의 gh-pages 브랜치를 생성해 푸시

    ```bash
    $ yarn deploy
    ```

  - github pages가 hosting 하고 있는 주소로 접속 가능

    `https://<github_id>.github.io/<repository>/`

    

## GitHub Actions workflow로 배포 자동화

> 위 과정처럼 수동으로 deploy하여 Github Pages에 수동으로 빌드된 정적파일을 github에서 호스팅 해줄 수도 있지만, github Actions를 통해 개발자가 새 소스 코드를 push하거나 pull request 같은 이벤트에 반응하여 트리거하도록 구성할 수 있음

##### Simple workflow

- Github Actions 메뉴의 simple workflow 파일을 커밋

- /.github/worflows/deploy.yml

  - jobs.deploy.steps: 실행할 sequence 명시
  - Actions를 통해 triggered 시 해당 step의 작업 순차 시행

  ```yaml
  # /.github/workflows/deploy.yml
  
  # A workflow run is made up of one or more jobs that can run sequentially or in parallel
  jobs:
    deploy:
      runs-on: ubuntu-latest
  
      steps:
        - name: Checkout source code
          uses: actions/checkout@master
        
        - name: Set up Node.js
          uses: actions/setup-node@master
          with:
            node-version: 14.x
        
        - name: Install dependencies
          run: yarn install
  
        - name: Build page
          run: yarn build
          env:
            NODE_ENV: production
  
        - name: Deploy to gh-pages
          uses: peaceiris/actions-gh-pages@v3
          with:
            github_token: ${{ secrets.GITHUB_TOKEN }}
            publish_dir: ./dist
  ```
  
- Issue(to do)

  - 위 코드로 deploy workflow를 실행하면 해당 페이지 이동 시 404 File not found가 출력

  

## 코드 수정 및 테스트 실패로 인한 자동 배포 실패 확인

> 빌드 자동화 시 테스트를 통과하지 못한 코드가 반영되지 않도록 테스트 유닛 script를 추가


