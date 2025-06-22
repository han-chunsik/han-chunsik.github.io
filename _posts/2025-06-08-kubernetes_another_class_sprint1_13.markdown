---
layout: post
title:  "배포 파이프라인 구축 - Argo"
date: 2025-06-22
categories: [kubernetes, container, devops]
description: "[Inflearn Warm-Up Club - DevOps] 매일 1% 성장하기"
comments: true
bird_image: "pionus.webp"
bird_name: "피어니스 (Pionus)"
bird_scientific_name: "Pionus"
bird_description: "피오니스 앵무는 중남미에 서식하는 중형 앵무새로, 차분한 성격과 상대적으로 조용한 소리로 잘 알려져 있다. 대표적으로 청머리피오니스는 파란 머리와 초록빛 몸통이 특징이며, 사육 난이도가 낮고 애완조로도 인기가 많다."
---

> [**쿠버네티스 어나더 클래스 (지상편) - Sprint 1, 2**](https://www.inflearn.com/course/%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4-%EC%96%B4%EB%82%98%EB%8D%94-%ED%81%B4%EB%9E%98%EC%8A%A4-%EC%A7%80%EC%83%81%ED%8E%B8-sprint1)

> 출처: [큐브옵스 - ArgoCD 빠르게 레벨업 하기 (1/3)](https://cafe.naver.com/f-e/cafes/30725715/articles/118?menuid=13&referrerAllArticles=false), [큐브옵스 - ArgoCD 빠르게 레벨업 하기 (2/3)](https://cafe.naver.com/f-e/cafes/30725715/articles/121?menuid=13&referrerAllArticles=false) , [큐브옵스 - ArgoCD 빠르게 레벨업 하기 (3/3)](https://cafe.naver.com/f-e/cafes/30725715/articles/125?menuid=13&referrerAllArticles=false) 
> 본 글은 출처를 참고하여 테스트를 진행한 결과를 바탕으로 작성하였습니다. 설치 및 실습 과정은 원문을 참고하시고, 이 글에서는 핵심 개념 정리와 추가 설명에 집중하였습니다.

# 배포 파이프라인 구축 실습 - Argo



# 회고
여러 가지 배포 파이프라인 구축 방법에 대해 개념을 정리하고, 실습을 통해 각 도구들의 필요성과 사용 목적을 체감할 수 있었다. 단순히 "이 도구를 쓰면 된다"가 아니라, 어떤 문제를 해결하기 위해 이 도구를 선택하게 되었는지, 각 단계에서 어떤 역할을 하는지를 이해하는 데 집중할 수 있었던 시간이었다.  
무엇보다도, 새로운 기술을 익힐 때 단순히 빠르게 구축해보는 것도 중요하지만, 직접 수동으로 구성해보면서 내부 동작을 이해하는 과정이 훨씬 큰 학습이 된다는 점을 다시금 느꼈다. 앞으로도 단순히 결과만 보지 않고, 그 과정과 맥락을 함께 공부하는 태도를 유지하고 싶다.