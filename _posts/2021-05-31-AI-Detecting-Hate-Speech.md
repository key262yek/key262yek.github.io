---
title: "How can AI detect hate speech"
date: 2021-05-31T21:35:30-22:00
categories:
  - Review
  - Sociology
tags:
  - AI
  - Hate speech
  - Unfinished
toc: true
toc_label: "Table of Contents"
toc_icon: "fas fa-clipboard-list"
toc_sticky: true
---

얼마전 TensorFlow KR 그룹에 [게시글](https://www.facebook.com/groups/TensorFlowKR/permalink/1492216937786026/?__cft__[0]=AZWtg5JcanG8KnfFNgyo5-uEQmYiE6ikM3Wg15A307Jw_QU-jlVznNcNbBpqK_lfzDzgH0gEqw23YKq4Dks3nOwJe5vkXLLLqZNwy0zl2hX2iB75u3bErj2iwDCBOYbbCw9uVSfWmWGw8yWkwgm7Pn8j9SoJCaAEUR5Gr5dWAJ1_zZHdGIU3el_E_UZY0SyCpV8&__tn__=%2CO%2CP-R) 하나가 공유됐다. 기존의 혐오발화 데이터셋 BEEP!를 이용해 혐오발언을 분류하는 모델 SoongsilBERT:BEEP!를 소개하는 글이다. 데모는 [https://bit.ly/3ucOL9p
](https://bit.ly/3ucOL9p
)에서 이용해볼 수 있다. 연구결과에 따르면 이 모델은 새롭게 만들어진 혐오표현이나, 특정 문화권에 관한 차별 발언들을 반영하지 못하는 한계가 있다고하며, 이를 해결하기 위한 방법으로 더 다양한 불특정 다수가 참여해 테스트 셋을 만드는 것을 제안했습니다. 

이에 관해 몇 가지 궁금한 점이 생겨 관련한 정보를 찾아 정리하기 위해 게시글을 열었습니다.
혐오와 차별은 발화된 발언만 가지고 판단하기 매우 어려운 특성을 가지고 있습니다.
같은 발언이라고 해도 발언자에 따라 달라질 수 있고(N word를 사용하는 흑인과 같은 경우), 일상적인 표현이더라도 맥락에 따라 차별적인 표현이 될 수도 있습니다. (부하 사원에게 맥락없이 "몸매가 좋네", "오늘 이쁘네" 같은 발언을 하는 경우)
발화자, 청자, 혹은 인용되는 개개인의 사회적 위치에 따라 혐오발언이 될 수도 있고, 아닐수도 있기 때문에 사회학에서 혐오와 차별을 연구하기 위해서 굉장히 다층적인 사회구조를 두루 살필 수밖에 없는 것입니다. 

따라서 혐오표현을 분류하는 AI 역시 이 다층적인 구조를 학습하지 않으면 당연히 한계를 가질 수밖에 없을 것이고, 짧은 문장들은 당연하고, 꽤 긴 문장들을 데이터셋으로 가지고 있더라도 제대로된 패턴을 찾을 수 없을테죠.
아마도 복잡계 물리에서 도시내 인프라를 분석하기 위해 버스, 지하철 등의 여러 network를 복합적으로 연결한 Multi-layer network과 같은 방식의 접근이 필요할 수 있겠다 싶습니다.
애초 학습 데이터와 layer를 복합적으로 갖춰두고 학습시킬 수도 있을테고, 개별적으로 사회구조, 발언에 대해 학습한 후에 이들을 선생으로 하여 병합한 AI를 만드는 방법도 생각해볼 수 있을 것 같습니다.

물론 AI에 대해 어깨넘어 들은 것이 전부인지라, 이미 이런 연구가 진행되고 있는지도 모르기 때문에 관련한 자료를 정리해보고자합니다.

### BEEP! Korean Corpus of Online News Comments for Toxic Speech Detection[^1]


[^1]:https://www.aclweb.org/anthology/2020.socialnlp-1.4/



