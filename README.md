
# 개요- 페르마타란?
> 페르마타는 블루투스(BLE)신호 기반의 접촉추적앱([contacts tracing app](https://en.wikipedia.org/wiki/COVID-19_apps))의 한종류로, 보다 빠른 **전파자 추적**이 가능한 프레임워크를 포함하고 있습니다. 기존의 접근방식이 확진자의 1차 접촉자들에게만 위험을 공유하는 것과 달리, **페르마타는 네트워크 모델 분석과 연쇄적인 위험정보 전파** 통해서 알려지지 않은 전파자(슈퍼전파자포함)와 그 접촉자들이 빠르게 진단검사를 받을 수 있도록 돕습니다. 

# BLE기반 접촉추적앱 동작방식
여러방식이 존재하지만, 기본적으로 블루투스 신호를 교환하는 것을 핵심으로 합니다.
![](https://i.imgur.com/d5X7uRM.png)

알림을 통해서 전파속도를 늦추는 효과가 있습니다.

![](https://i.imgur.com/atVtSs3.png)

기본 동작을 기본으로 나라마다 [다양한 운영방식이 적용](https://en.wikipedia.org/wiki/COVID-19_apps#List_of_apps_by_country)되고 있습니다. 


# 현재까지 방식의 한계
 
> 빠르게 확장하는 네트워크가 현실이지만, 이를 반영하지 못하고 속도를 따라가지 못함.

다루는 관계가 단편적인 개인간의 1:1 연결이기 때문에, 현실을 반영하는데 한계가 있습니다. 발견한 확진자를 마치 유일한 전염원인것처럼 다루고 있지만, 실제로는 감염자-네트워크의 일부일 뿐입니다. 발견한 확진자를 네트워크의 root로 생각하는 것은 편의적으로 그렇게 하는 것일뿐, 실제로는 누가 n차이고 n+1차인지는 알수가 없습니다.  이러한 한계는 장님코리끼 만지는 식의 분석과 불충분한 실행을 야기합니다. 

추가: [영국은 연쇄적인 알림기능을 도입했습니다.](https://www.bbc.com/news/technology-52441428) 이를 위해서 구글/애플 방식을 포기했습니다. 이 방식으로는 중앙에서 수집하는 것이 어렵다고 보는듯 합니다. (접촉정보를 모두 서버로 수집하는 것은 fermata 베타버젼에서 구현한 방식과 비슷합니다.)



# 페르마타의 구현 및 운영 방식
페르마타는 2가지 기능을 통해서, 빠르고 효율적으로 전파대상의 범위를 정하고, 위험을 공유합니다. 

## 1. 연쇄적인 위험정보 전파
빠른 검사는 감염자를 줄이는데 아주 중요한 요소라는 것이 증명되었습니다.페르마타는 감염위험을 가능한 빠르게 알리기 위해서, 확진자의 접촉자 뿐아니라, 접촉자의 접촉자들까지도 알림대상으로 취급합니다. 이를 구현하기 위해서, 확진자폰에 저장되어 있는 접촉정보 뿐 아니라, 접촉알림을 받은 폰에 저장된 접촉정보도 사용하게 됩니다. 이런 식으로 연쇄적으로 진행하면, 확진자 발생 즉시, 감염가능성이 있는 네트워크 전체를 커버할 수 있습니다.기존 방식으로 n단계 알림은 검사확인소요시간(3-4일)의 n배가 걸리는 일입니다. 검사하고, 2일 대기하고, 알리고, 알림받고,3일 대기하고  알리고, 이러한 긴 대기시간을 제거할 수 있습니다. 몇단계까지 적용하는가는 뒤에 다루는 네트워크 분석을 통해서 정하게 됩니다. 


## 2. 지능적인 네트워크 분석 및 탐색 
페르마타는 확진자 폰에 저장되어 있는 접촉자들 리스트뿐 아니라,  네트워크 구조를 분석합니다.단순한 접촉자를 찾는 것이 아니라, 전파방향을 분석하여, 시작점(root)를 향하여 탐색을 진행시킵니다. 

단순화 시킨 예를 들면, 확진자A의 폰에 기록된 접촉자가 1명(B)뿐 인 경우를 생각해볼수 있습니다.
이경우, 확진자A가 B에게 전파한것이 아니라, B가 A에게 전파했을 확률이 훨씬 큽니다. A가 바이러스를 만들어내지 않는 이상, A도 누군가에게 받은 것이고, 접촉자가 1명이라면, 바이러스를 전달해준 사람은 바로 B입니다. B가 아직 검사전이지만, 확진자일 확률이 높다는 것을 미리 알수 있습니다. 그러므로 B에게 검사받으라고 알려줄 뿐아니라, B의 접촉자들에게 위험을 반드시 알려줘야 합니다. 
B의 폰이 알림을 받으면, 같은 방식으로 분석을 합니다. 만약, 이번에는 접촉자가 2명(C,D)이라면? C,D는 50%의 확률로, 전파자일 것입니다. 이런 상황에서 D에게 다른 접촉자가 없다면,D보다는 C가 전파자일 확률이 커집니다. 

완전히 상반된 상황도 있을 수 있습니다. 기록된 접촉자가 짧은 시간에 수백명에 이른다면, 대량 전파의 가능성을 열어두고, 3,4단계를 거친 모든 사람에게 위험을 알려야 합니다. 
이러한 네트워크분석은 확률과 통계를 기반으로하는 알고리즘을 통해서 서버상에서 구현이 될 것입니다.

이상적인 경우라면, 기존에 파악했던 확진자와 연결이 될떄, 이러한 탐색이 종료됩니다. 즉,깜깜이 전파가 아니라,측정 가능하고 관리가능한 대상으로 변하게 됩니다. 
이는 알림을 지능적으로 만들뿐 아니라, 실제 역학조사의 효율성을 극대화하는데 중요한 역할을 하게 됩니다.

# 서비스 구현 
## 앱(클라이언트)
구글/애플이 제시한 BLE기반 접촉알림앱의 기본적인 기능은 그대로 사용하게 됩니다. 추가적으로 다음과 같은 기능이 필요합니다. 
### 접촉알림 수신시 접촉정보 전송
* 기본 앱은 접촉알림을 받으면 그것으로 끝이지만, 페르마타는 이 시점에서 해당기기(확진자의 접촉자의 기기)의 접촉기기들을 찾는 작업을 더 진행할 수 있습니다. 
* 이를 위해서 앱설치시 이러한 과정에 대한 동의가 필요함.
* google/apple 방식 위에서 구현하기 때문에, 영국과 같이 접촉정보를 완전히 수집하지는 않음..

## 서버
1차적인 알림을 주는 기본적인 알림기능과, 알림시 수집되는 접촉기록들을 네트워크로 구성하여 분석하는 역할도 포함하고 있음. 