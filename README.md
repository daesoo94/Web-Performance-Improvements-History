# Web-Performance-Improvements-History
Last Updated : (2018.12.20)

회사에서 웹 퍼포먼스 개선 작업을 맡게 되면서, 작업한 내용을 Documentation 하여 History로 작성할 예정이다.

# Done
1. Gzip
  기본적으로 우리 프로젝트는 gzip이 자동으로 html문서에 적용되어서 날라가지만, js, css같은 문자기반 리소스들은 그냥 날라간다.
  (다행히 compression, uglify 는 되어있다. production mode니까 당연하겠지만..)
  
  이 부분은 네트워크 성능에 따라서 너무나도 심각한 속도차이를 유발하였고, Performance test에서 문제가 되었다.
  따라서 gzip 효율이 좋은 js, css를 gzip 압축하여 전송하게 될 경우 조건부에 성능적으로 우수해진다. (조건 : 네트워크 다운속도로 보는 이득 > 디코딩 시간)
  
2. Font,CSS Preload
  Font, CSS의 경우 미리 Preload하여 초기에 빠르게 받아두어, 스타일이 적용되도록 한다.
  Font의 경우 Web Font를 사용하지 않으면 다운할 일이 없으나, Web Font를 사용하고 있기에, 미리 사용하는 폰트에 대해서는 Preload해둔다.
  (웹 폰트의 경우 한글 웹폰트는 300kb쯤 되는데, 이것을  5개나 Load 해서 사용중이다.. 이 부분은 상당히 마음에 걸리나, 내 영역은 아니고 퍼블리셔와 이야기해 보아 
  야할 것이다. 총 다운받는데 걸리는 시간이 3G 기준으로 압도적으로 크다. 초기 병렬 다운받을 수 있는 리소스도 제한되어 있으니.. 추후 이야기 할것)
  
3. Image Lazy-Load, Load
  기본적으로 우리 프로젝트는 약 30개 이상의 Image Resources를 로드하여 사용한다.
  (chart를 그리는 것보다, 작은 크기의 chart images를 만들어두고 불러와서 paint하는것이 싸기 때문이다.)
  이 때, 네트워크 자원을 상당히 많이 사용하는데, 현재 보이지 않는 chart images에 대해서는 lazyload를 사용하여, 자원 소모를 막으며,
  모든 항목에 lazy load를 사용하게 되었을 경우, 사용자가 페이지 방문 시, 항목이 늦게보이게 되기 때문에 사용자 만족도가 떨어질 것이다.
  따라서, 보이지 않는 항목에는 lazy-load를 사용하며, 보이는 항목에는 기본적인 load를 사용하여 사용자 만족도를 높이고자 한다.
  (또한 초기 paint점수를 많이 받을 수 있다고 판단 되었다.)
  
4. Code Splitting ( Webpack 4 )
  우리 프로젝트는 code splitting이 되어있지 않은 완벽한 하나의 bundle을 가지고 사용하고 있었는데.. 이 부분에 대해서 js파일을 route의 내용에 따라
  동적 import한 내용을 webpack의 code splitting 유도해서 페이지마다 여러개의 js파일로 나누었을 경우 상당히 좋을것이라는 기대와 함께 진행하였으나,
  결과물로서는 상당히 만족스럽지는 못하다. 그 이유는 Bundle Analyzer 결과를 참고하면 알 수 있다.
  
5. Tree Shaking ( Webpack 4 )
  또한 사용하지 않는 외부 의존성 모듈 (Dependencies)들을 제거하고자 Tree Shaking 을 웹팩으로 시도해 보았으나, 추후 찾아본 바로는, Webpack 4 에서는
  Production Mode 에서는 기본적으로 Treeshaking을 해주는 것 으로 알고있다. 음.. js 파일의 크기가 상당히 큰데.. 이 부분을 줄여서 초기 로드 속도를
  높이고 싶으나.. 아직 쉽지 않다.. 이 부분도 공부좀 할 것.
  
6. Bundle Analyzer ( Webpack 4 )
  우리 프로젝트의 Bundle을 분석하여, 사용하지 않는 모듈, 어떤 Dependency가 용량을 많이 차지하는가? 등을 분석하기 위해서 Webpack Bundle Analyzer를
  추가하고, 사용해 보았다. 결과적으로 먼저 말하자면, App.js Bundle에서는 ChartIQ가 차지하는 용량이 거의 반절로, 압도적이라 할 수 있다. 물론 전문적인
  Chart를 지원해야 하기 때문에 ChartIQ를 포기할 수 없으므로, 다른 Dependency를 가벼운 것으로 바꾸거나 해야겠지만.. ChartIQ에서 Javascript도 사용하는데
  이 또한 양이 상당하고.. Dependency를 줄일 수 있는것이 딱히 보이지 않는다.. ( 이 부분도 어떻게 해야 js를 줄일 수 있을지 생각해 보아야 할 것 이다.)

# To Do

1. Images Optimize
  현재 우리는 News를 크롤링 하여 보여주게 된다. 이 때 News의 thumbnail images들을 가져오는데, 이 images들의 크기가 천차 만별이다. 또한 사이즈가
  맞지 않을 경우 -> down하고 보여주지도 않는다!!.. 따라서.. back-end에서 images를 형식에 맞게 수정하여 s3에 저장하여 사용하면 좋을 것 같은데..
  이 과정이 얼마만큼의 자원을 소모하는지 자세하게 알지 못하니 이 또한 일단 가만히 둔다.
  
2. 위에서 하지못한 js 줄이기
  2가지의 경우를 두어본다.
  
  2.1 네트워크 속도가 빠를때.
    이 때의 경우 js파일의 evoluate 시간이 900ms 정도로 압도적으로 크다. (전체 page load : 1.8 ~ 2 sec)
    따라서 1초대의 page load를 원하는 현재, js 의 압도적인 크기는 너무나도 절망적이다. 따라서 이 js파일을 어떻게하면 줄일 수 있을지 계속하여 생각하고,
    analyzer의 내용을 참고하여 제거, 또는 변경을 가져야 할 것 같다.
    
  2.2 네트워크 속도가 느릴때.
    이 때의 경우 js파일의 evoluate 시간인 900ms는 크지 않다. 총 page load 시간 8 sec 나 되는데 이 때의 경우 resource down시간이 압도적으로 크다.
    결과적으로 resource를 줄이고 줄였으나, 아직도 많다는 결론이며, image optimize를 거쳐서 resource 크기를 최대한 줄여야 할 것이다.
    또한 web font를 사용해야 하는가? => (미적으로 중요한가, 빠른 응답성이 중요한가?) 에 대한 이야기를 진행하여 해결해야할 것 같다.

3. 잘 알면 최적화가 보인다. webpack공부와 추가적 front-end 공부
  당연하게도 많이 아는만큼 최적화 할 수 있을 것이다. 내가 진행한 최적화는 아주 기본적인 최적화들 이기 때문에 심오한 깊이의 최적화는 물론 진행하지 못했다.
  지금도 공부하고, 앞으로도 공부하겠지만, 좋은 예제들을 좀 잘 찾아서 최적화 하는 방식을 찾아 보아야 겠다.. ( 무슨 시말서 같다. )
