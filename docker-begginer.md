# DevOps

 Vue Cli 프로젝트를 기반으로 DevOps 개발환경을 다루는 실습을 하였다.

Vue.js는 기본적인 프로젝트 구조를 빠르게 스케풀딩 할 수 있는 공식 CLI를 제공한다. 이 것을 사용하여 프로젝트를 생성하고 Github와 연동하고 Github Pages라는 서비스에 정적 페이지를 호스팅하도록 배포(Deploy)한다. 또한 소스코드를 커밋 & 푸시 만으로 자동화 할 수 있는 워크플로우를 구성하여 Github Actions를 통하여 배포 자동화를 했다. 테스트 성공 여부에 따라 배포 자동화가 잘 작동하는지 확인까지 완료하였다.



## 1. 사전 환경 구성

https://classic.yarnpkg.com/en/docs/install 에서 yarn을 설치 한다.

다음 명령어로 Vue CLI를 설치하여 버전을 확인해 본다.

```
C:\>yarn global add @vue/cli
C:\>vue -V
@vue/cli 4.5.13
```



## 2. Vue 프로젝트 생성

Git 정보 설정을 해준다.

```
git config --global user.name "김낙현"
git config --global user.email krh963852@daum.net
```

프로젝트 생성 과정에서 아래와 같은 순서로 선택한다.

Manual select features, Unit Testing 추가 선택, 3.x, ESLint + Prettier, Lint on save, Jest, In dedicated config files, N

```
C:\dev>vue create vue-devops
```



## 3. 웹브라우저에서 확인

```
C:\dev> cd vue-devops
C:\dev\vue-devops>yarn serve
```

Ctrl + c로 서버를 종료한다.



## 4. 깃허브에 push 및 pages에 수동 배포

New repository 생성.

Repository name : vue-devops	/	public으로 생성.



원격 저장소 설정 및 코드 푸시.

```
C:\dev\vue-devops> git remote add origin https://github.com/HelloNaks/vue-devops.git
C:\dev\vue-devops> git push -u origin main
```



package.json에서 배포에 필요한 정보 추가

```
C:\dev\vue-devops> yarn add gh-pages -D
```

```
{
  "name": "vue-devops",
  "version": "0.1.0",
  "private": true,
  "homepages": "https://hellonaks.github.io/vue-devops",
  "scripts": {
    "serve": "vue-cli-service serve",
    "build": "vue-cli-service build",
    "predeploy": "vue-cli-service build",
    "deploy": "gh-pages -d dist",
    "clean": "gh-pages-clean",
    "test:unit": "vue-cli-service test:unit",
    "lint": "vue-cli-service lint"
  },
  "dependencies": {
    "core-js": "^3.6.5",
    "vue": "^3.0.0"
  },
  "devDependencies": {
    "@vue/cli-plugin-babel": "~4.5.0",
    "@vue/cli-plugin-eslint": "~4.5.0",
    "@vue/cli-plugin-unit-jest": "~4.5.0",
    "@vue/cli-service": "~4.5.0",
    "@vue/compiler-sfc": "^3.0.0",
    "@vue/eslint-config-prettier": "^6.0.0",
    "@vue/test-utils": "^2.0.0-0",
    "babel-eslint": "^10.1.0",
    "eslint": "^6.7.2",
    "eslint-plugin-prettier": "^3.3.1",
    "eslint-plugin-vue": "^7.0.0",
    "gh-pages": "^3.2.3",
    "prettier": "^2.2.1",
    "typescript": "~3.9.3",
    "vue-jest": "^5.0.0-0"
  }
}

```



vue.config.js 파일 생성.

```
module.exports = {
  publicPath: "/vue-devops/",
  outputDir: "dist"
};
```



yarn deploy 명령을 실행하면 빌드된 정적 파일을 원격 저장소의 gh-pages 브랜치를 생성해서 푸시된다.

```
C:\dev\vue-devops>yarn deploy
```

프로젝트의 Settings에 pages 부분에 링크가 잘 입력되었는지 확인.



repository에서 사이트로 쉽게 이동하도록 웹사이트 주소 설정.

우측 상단의 About 클릭 후 링크 삽입.



## 5. 깃허브 Actions workflow로 배포 자동화

Action를 클릭한다.

Simple Workflow를 생성하고 deploy.yml로 이름짓는다.

내용 중 name에 Deployment를 삽입 후 commit을 한다.

(커밋과 동시에 workflow가 동작된다.)



VS Code에서 좌측 하단에 Sync버튼을 누른다.

deploy.yml파일을 아래와 같이 수정해준다.

```
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

        - name: Test unit
          run: yarn test:unit

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

변경된 파일들을 Stage에 추가해서 커밋 & 푸시를 해준다.



## 6. 코드 재 수정 및 배포 성공 확인

deploy.yml에서 yarn install 과 build 사이에 test unit 단계를 추가한다.

```
- name: Test unit
  run: yarn test:unit
```

실패하면 데이터가 넘어가지 않는 것을 확인할 수 있다.



## 7. 코드 수정 및 테스트 실패로 인한 자동 배포 실패 확인

다시 변경하여 잘 돌아가는지 확인한다.



## 총 정리

Vue CLI 프로젝트 기반 DevOps 개발환경 실습을 하면서 VS Code를 사용하여 github에 연동하고 바로 올라가는 것이 아닌 테스트를 거쳐서 문제가 없을 때 서버에 올라가도록 하는 것을 배웠다. 싸피에서 첫 번째 프로젝트를 시작하게 될 텐데 이 기술을 활용하여 팀원들과 조금 더 빠르게 정확한 코드들을 주고 받을 수 있게 될 것 같다고 생각한다.



## 기여자


 <td align="center"><a href="https://github.com/HelloNaks"><img src="https://avatars.githubusercontent.com/u/49478141?v=4?s=100" width="100px;" alt=""/><br /><sub><b>HelloNaks</b></sub></a><br /><a href="#platform-HelloNaks" title="Packaging/porting to new platform">📦</a></td>