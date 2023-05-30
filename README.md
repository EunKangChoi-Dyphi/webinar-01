<br/>
<br/>

<p align="center">
<img src="https://files.cloudtype.io/logo/cloudtype-logo-horizontal-black.png" width="50%" alt="Cloudtype"/>
</p>

<br/>
<br/>

# 클라우드타입 웨비나 #01 <br/> GitHub Actions를 활용한 CI/CD 파이프라인 구축하기 <!-- omit in toc -->

## 목차 <!-- omit in toc -->
- [🛠️ 준비사항](#️-준비사항)
- [⚙️ GitHub Actions Workflows](#️-github-actions-workflows)
  - [배포된 React 앱에 GitHub Actions 적용하기](#배포된-react-앱에-github-actions-적용하기)
  - [프라이빗 레지스트리 이미지 배포하기 - GitHub Container Registry](#프라이빗-레지스트리-이미지-배포하기---github-container-registry)
- [📖 References](#-references)
- [💬 Contact](#-contact)

## 🛠️ 준비사항
- [GitHub 계정](https://github.com/)
- [클라우드타입 계정](https://cloudtype.io/)
- [예제 소스 GitHub 저장소](https://github.com/cloudtype-examples/webinar-01)

## ⚙️ GitHub Actions Workflows

### 배포된 React 앱에 GitHub Actions 적용하기
<img width="1171" alt="스크린샷 2023-05-30 오후 2 45 22" src="https://github.com/EunKangChoi-Dyphi/webinar-01/assets/129829722/3c867f0e-87ed-41e6-ab3d-79e41512a62f">
<img width="1163" alt="스크린샷 2023-05-30 오후 2 30 13" src="https://github.com/EunKangChoi-Dyphi/webinar-01/assets/129829722/41d77a91-5550-47d5-b231-e1cc6797b3b0">


```yaml
name: Deploy to cloudtype
on:
  push:
    branches:
      - main
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2
      - name: Connect deploy key
        uses: cloudtype-github-actions/connect@v1
        with:
          token: ${{ secrets.CLOUDTYPE_TOKEN }}
          ghtoken: ${{ secrets.GHP_TOKEN }}
      - name: Deploy
        uses: cloudtype-github-actions/deploy@v1
        with:
          token: ${{ secrets.CLOUDTYPE_TOKEN }}
          project: [스페이스명]/[프로젝트명]
          stage: main
          yaml: |
            name: [서비스명]
            app: web
            options:
              docbase: /build
              nodeversion: 14
              spa: true
            context:
              git:
                url: git@github.com:${{ github.repository }}.git
                ref: ${{ github.ref }}
              preset: react
```

### 프라이빗 레지스트리 이미지 배포하기 - GitHub Container Registry

<img width="1167" alt="스크린샷 2023-05-30 오후 2 45 29" src="https://github.com/EunKangChoi-Dyphi/webinar-01/assets/129829722/97944221-46f4-4762-abeb-5309503f0432">


```yaml
name: Create and publish a Docker image, Deploy to Cloudtype

on:
  push:
    branches:
      - main

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build-and-push-image:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write

    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Log in to the Container registry
        uses: docker/login-action@65b78e6e13532edd9afa3aa52ac7964289d1a9c1
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GHP_TOKEN }}

      - name: Extract metadata (tags, labels) for Docker
        id: meta
        uses: docker/metadata-action@9ec57ed1fcdbf14dcef7dfbe97b2010124a938b7
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=sha

      - name: Build and push Docker image
        uses: docker/build-push-action@f2a1d5e99d037542a71f64918e516c093c6f3fc4
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}

      - name: Deploy to Cloudtype
        uses: cloudtype-github-actions/deploy@v1
        with:
          token: ${{ secrets.CLOUDTYPE_TOKEN }}
          project: [스페이스명]/[프로젝트명]
          stage: main
          yaml: |
            name: [서비스명]
            app: container
            options:
              ports: 8080
              image: ${{ steps.meta.outputs.tags }}
```


## 📖 References

- [클라우드타입 Docs](https://docs.cloudtype.io/)

- [클라우드타입 FAQ](https://help.cloudtype.io/guide/faq)
  
- [GitHub Actions Docs](https://docs.github.com/en/actions)
  
## 💬 Contact

- [Discord](https://discord.gg/U7HX4BA6hu)
