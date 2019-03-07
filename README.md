# Unit Testing in ASP.NET Core Web API  

Examples included in the blog post: https://code-maze.com/unit-testing-aspnetcore-web-api/


# 주의 사항

코드를 다운받아 실행시키면, 아래와 같은 메세지가 출력되면서 테스트가 안될 수 있다. 

로그 정보 : ======== 테스트 실행 완료: 0개 실행 =========

Nuget에서 Microsoft.AspNetCore.App 설치할때, 
대상 프로젝트의 Microsoft.AspNetCore.App과 버전이 달라서(낮아서) 그럴 수 있다.
xUnit 프로젝트의 Microsoft.AspNetCore.App을 삭제하고, 대상 프로젝트와 같은 버전을 설치해주면 된다.
