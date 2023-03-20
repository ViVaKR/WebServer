### HTTPS  

> $ dotnet tool install -global win-acme  
>> run $ wacs  
>> [Download Site]('https://www.win-acme.com/')
---

### URL ReWrite  
> 설정  
1. 정규식 (.*) : 프로토콜, 호스트명, 포트를 제외한 모든 형식의 URL
	e.g. https://www.vivabm.com:8080/product/price.html
	      에서 패턴매치 되는 부분은 product/price.html 부분

2. {HTTPS}, 패턴과 일치, ^OFF$  
> - IIS 서버 변수 {HTTPS} 
> - https 요청이면 ON   
> - http 요청이면 OFF  
> - 즉, 본 설정은 http 요청인 경우만 처리.

1. 서버 변수, 리디렉션
> - https://{HTTP_HOST}/{R:1}

1. 리디렉션 유형, 영구(301)  
> - 리다이렉트 될 URL 생성  
> - IIS 서버변수 {HTTP_HOST} : 도메인명  
> - 포트 변경시 : https://{HTTP_HOST}:1234/{R:1}  
> - 정규식 : {R:1}은 그룹에 대한 역참조 입니다. 그룹은 소괄호로 둘러싼 부분  
> - 정규식 (.*) : 하나의 그룹  
> - {R:0} : 전체 입력 문자열을 나타냄   
> - {R:1} : 첫번째 그룹입니다  
> - 정규식 (.*) 에서 {R:0}와 {R:1}은 같은 값을 표시함  