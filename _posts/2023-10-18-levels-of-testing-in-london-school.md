---
layout: post
title: 테스트 수준 (통합 테스트란 무엇인가)
categories: [스터디-테스트]
tags:
  [
    테스트,
    테스트 수준,
    단위 테스트,
    통합 테스트,
    인수 테스트,
    목,
    테스트 대역,
    end-to-end,
    고전 스타일,
    런던 스타일,
  ]
date: 2023-10-18 13:00:00 +0900
---

Growing Object-Oriented Software, Guided by Tests  
by Steve Freeman, Nat Pryce  
ch1. Levels of Testing

---

고전 스타일과 런던 스타일에 대해서는 아래 글에서 정리해두었다.  
[단위 테스트의 두 분파 (고전파와 런던파)](/2023/10/05/%EB%8B%A8%EC%9C%84-%ED%85%8C%EC%8A%A4%ED%8A%B8%EC%9D%98-%EB%91%90-%EB%B6%84%ED%8C%8C)

런던 스타일에 대해서는 Steve Freeman와 Nat Pryce의 Growing Object-Oriented Software, Guided by Tests 를 추천해주었기 때문에 '런던 스타일의 입장에서 통합 테스트란 무엇을 의미하는가' 를 정리하고 싶어서 이 책의 내용을 조금 따와서 번역하고 정리해보았다.
내용은 길지 않으나 검색으로는 명확한 설명이 보이지 않았기에 글을 작성한다.

---

# 테스트 수준

- **인수 테스트 (Acceptance Test)** : 전체 시스템이 제대로 작동하나요?
- **통합 테스트 (Integration Test)** : 우리 코드는 우리가 변경할 수 없는 코드에 대해서 제대로 작동하나요?
- **단위 테스트 (Unit Test)** : 우리 객체가 제대로 동작하나요? 편리하게(convenient) 동작하나요?

TDD 세계에서는 "기능 테스트(functional test)", "고객 테스트(customer test)", "시스템 테스트" 등의 인수 테스트라고 부르는 용어에 대해 많은 논의가 있었습니다. 아쉽게도 인수 테스트의 정의가 종종 통일되지 않는 경우가 있습니다.

인수 테스트는 도메인 점문가와 함께 다음에 구축할 내용을 이해하고 합의하는 데 도움을 주기 위해서 하는 것이며, 또한 개발을 계속하면서 기존 기능이 손상되지 않았는지 확인하기 위해 사용합니다.
인수 테스트의 역할(role)에 대한 선호되는 구현 방식은 가능하면 end-to-end 테스트로 작성하는 것입니다.

우리는 통합 테스트라는 용어를 사용하여 일부 코드가 변경할 수 없는 팀 외부의 코드와 어떻게 작동하는지 확인하는 테스트를 나타냅니다. 영속성 매퍼(persistence mapper)와 같은 공개 프레임워크일 수도 있고 조직 내 다른 팀의 라이브러리일 수도 있습니다. 통합테스트를 통해 타사 코드를 기반으로 구축한 추상화가 예상대로 작동하는지 확인합니다.
통합 테스트를 통해 외부 패키지의 구성 문제를 파악하는데 도움이 되고 인수 테스트 보다는 더 빠른 피드백을 제공할 수 있습니다.

---

**고전 스타일**의 경우 테스트 대역을 최대한 쓰지 않으려 노력한다.
공유 의존성에 대해서만 테스트 대역을 사용한다. 또한 외부 의존성 중에서도 우리가 관리할 수 없는 영역에 대해서만 테스트 대역을 사용한다.([언제 목을 써야 할까](/2023/09/14/8장-통합-테스트를-하는-이유-2) 참조)
단위 테스트는 단일 동작 단위를 검증하고, 빠르게 수행 가능해야 하고, 다른 테스트와 별도로 처리할 수 있어야 한다. 목은 통합 테스트에서만 사용해야한다. ([목 처리에 대한 모범 사례](/2023/09/21/9장-목-처리에-대한-모범-사례) 9.2.1 장 참고)

**런던 스타일**의 경우 목(mock)을 고전 스타일에 비교했을 때는 더 적극적으로 사용한다. 여기서 오늘 정리한 내용을 통해서 보았을 때 단위 테스트와 통합 테스트의 차이는 테스트 대역을 사용한 부분이 외부 의존성인지 아닌지의 여부로 나눠지는 것으로 보인다.