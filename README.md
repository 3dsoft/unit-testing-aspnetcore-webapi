# Unit Testing in ASP.NET Core Web API  

Examples included in the blog post: https://code-maze.com/unit-testing-aspnetcore-web-api/


# 주의 사항

# 테스트 대상 프로젝트를 참조 추가

# Nuget에서 Microsoft.AspNetCore.App 설치할것 (버전 확인할것!!!)

  * 대상 프로젝트의 Microsoft.AspNetCore.App과 동일한 버전을 설치해야한다.
  
    버전이 다르면 테스트 함수를 못 찾아서 테스트가 안됨


# xUnit 클래스의 생성자를 만들고, 테스트할 Controller의 필드 멤버와 테스트할 Fake 클래스를 선언해준다.

'''
private readonly ShoppingCartServiceFake _service;
private readonly ShoppingCartController _controller;

public ShoppingCartControllerTest()
{
    // xUnit 프로젝트에 만들어둔 Fake 클래스 
    _service = new ShoppingCartServiceFake();

    // 테스트할 Controller 클래스
    _controller = new ShoppingCartController(_service);
}
'''


# Assert.IsType()을 사용해서 체크하려면 Controller의 return 타입을 ObjectResult 형식으로 명확히 지정해줘야한다.

아래처럼 return하면 xUnit에서 반환받았을때 객체 타입이 null이된다.

'''
[HttpGet]
public ActionResult<IEnumerable<string>> Get()
{
    return new string[] { "value1", "value2" };
}
'''

xUnit에서 Assert.IsType()을 받으려면 아래와 같이 해줘야한다.

'''
[HttpGet]
public ActionResult<IEnumerable<string>> Get()
{
    return Ok(new string[] { "value1", "value2" });
}
'''  
  
  
# Get 테스트 방법

[Fact]
public void Get_WhenCalled_ReturnsOkResult()
{
    var okResult = _controller.Get();

    // 리턴받은 결과가 OkObjectResult와 같으면 성공
    Assert.IsType<OkObjectResult>(okResult.Result);
}

[Fact]
public void Get_WhenCalled_ReturnsAllItems()
{
    var okResult = _controller.Get().Result as OkObjectResult;

    var items = Assert.IsType<List<ShoppingItem>>(okResult.Value);

    // 주어진 두 값이 같으면 성공
    Assert.Equal(3, items.Count);
}


[Fact]
public void GetById_UnknownGuidPassed_ReturnsNotFoundResult()
{
    // 랜덤한 ID 값을 입력한다.
    var notfoundResult = _controller.Get(Guid.NewGuid());

    // 결과가 NotFoundResult 형식이면 성공
    Assert.IsType<NotFoundResult>(notfoundResult.Result);
}

[Fact]
public void GetById_ExistingGuidPassed_ReturnsOkResult()
{
    var testGuid = new Guid("ab2bd817-98cd-4cf3-a80a-53ea0cd9c200");

    // 존재하는 ID 값을 넣어서 결과를 리턴받는다.
    var okResult = _controller.Get(testGuid);

    // 결과가 OkObjectResult 형식이면 성공
    Assert.IsType<OkObjectResult>(okResult.Result);
}

[Fact]
public void GetById_ExistingGuidPassed_ReturnsRightItem()
{
    var testGuid = new Guid("ab2bd817-98cd-4cf3-a80a-53ea0cd9c200");

    // 존재하는 ID 값을 넣어서 OkObjectResult 형식의 결과를 리턴받는다.
    var okResult = _controller.Get(testGuid).Result as OkObjectResult;

    // 리턴받은 값의 Value 형식이 ShoppingItem 형식이면 성공
    Assert.IsType<ShoppingItem>(okResult.Value);

    // 리턴받은 값의 Value에서 Id 값이 testGuid와 같으면 성공
    Assert.Equal(testGuid, (okResult.Value as ShoppingItem).Id);
}



# Post 테스트 (add)

[Fact]
public void Add_InvalidObjectPassed_ReturnsBadRequest()
{
    // 새로 추가할 아이템 정의
    var nameMissingItem = new ShoppingItem()
    {
        Manufacturer = "company",
        Price = 12.00M
    };

    // 새로운 아이템에 Name이없는데, 이 값이 없어도 OkRequestObjectResult가 됨.
    // BadRequestObjectResult 결과를 받기 위해 ModelState의 Name이라는 속성에 [Required]라는 어트리뷰트를 설정한다.
    _controller.ModelState.AddModelError("Name", "Required");

    // Post 형식으로 새로운 아이템 추가
    var badResponse = _controller.Post(nameMissingItem);

    // 결과가 BadRequestObjectResult 형식이면 성공
    Assert.IsType<BadRequestObjectResult>(badResponse);
}

[Fact]
public void Add_ValidObjectPassed_ReturnsCreatedResponse()
{
    ShoppingItem testItem = new ShoppingItem()
    {
        Name = "test Item",
        Manufacturer = "Company",
        Price = 15.00M
    };

    // Post형식으로 테스트 아이템을 보낸다.
    var createdResponse = _controller.Post(testItem);

    // 결과가 CreatedAtActionResult와 같으면 성공
    Assert.IsType<CreatedAtActionResult>(createdResponse);
}

[Fact]
public void Add_ValidObjectPassed_ReturnedResponseHasCreatedItem()
{
    var testItem = new ShoppingItem()
    {
        Name = "box",
        Manufacturer = "company",
        Price = 17.00M
    };

    // 신규 아이템을 입력하고 결과를 받는다.
    var createdResponse = _controller.Post(testItem) as CreatedAtActionResult;
    // 결과에서 Value 값만 가져온다.
    var item = createdResponse.Value as ShoppingItem;

    // 결과가 ShoppingItem 형식이면 성공
    Assert.IsType<ShoppingItem>(item);

    // Value 값에서 Name이 box와 같으면 성공
    Assert.Equal("box", item.Name);
}


# Delete 테스트

[Fact]
public void Remove_NotExistingGuidPassed_ReturnsNotFoundResponse()
{
    // 랜덤한 Guid 값을 생성해서 Delete 형식으로 넘긴다.
    var notExistingGuid = Guid.NewGuid();
    var badResponse = _controller.Delete(notExistingGuid);

    // 결과가 NotFoundResult면 성공
    Assert.IsType<NotFoundResult>(badResponse);
}


[Fact]
public void Remove_ExistingGuidPassed_ReturnsOkResult()
{
    // 이미 존재하는 Guid 값을 넘겨서 결과를 리턴받는다.
    var existingGuid = new Guid("ab2bd817-98cd-4cf3-a80a-53ea0cd9c200");
    var okResponse = _controller.Delete(existingGuid);

    // 결과가 OkResult면 성공
    Assert.IsType<OkResult>(okResponse);
}

[Fact]
public void Remove_ExistingGuidPassed_RemovesOneItem()
{
    // 존재하는 Guid 값을 넘겨서 삭제
    var existingGuid = new Guid("ab2bd817-98cd-4cf3-a80a-53ea0cd9c200");
    var OkResponse = _controller.Delete(existingGuid);

    // 삭제 후, 실제 Item 개수가 2개이면 성공
    Assert.Equal(2, _service.GetAllItems().Count());
}






