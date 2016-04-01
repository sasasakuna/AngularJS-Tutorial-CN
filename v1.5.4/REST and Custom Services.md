在这一步中，我们将改变应用获取数据的方式。
  * 我们定义了一个自定义服务代表了一个RESTful客户。使用这个客户我们能以一种比较简单的方式发送请求到服务器去获取我们的数据，而不需要去处理低级的$http API，HTTP方法和URLs。

重要的改变如下所示。你可以通过[GitHub](https://github.com/angular/angular-phonecat/compare/step-10...step-11)查看差异

---

### 依赖

RESTful的功能是在Angular的ngResource模块中定义的，该模块和核心Angular框架分开发布。

我们使用[Bower](http://bower.io/)安装客户端依赖。这一步将添加新的依赖到`bower.json`的配置文件中：
```
{
  "name": "angular-seed",
  "description": "A starter project for AngularJS",
  "version": "0.0.0",
  "homepage": "https://github.com/angular/angular-seed",
  "license": "MIT",
  "private": true,
  "dependencies": {
    "angular": "1.4.x",
    "angular-mocks": "1.4.x",
    "jquery": "~2.2.1",
    "bootstrap": "~3.1.1",
    "angular-route": "1.4.x",
    "angular-resource": "1.4.x"
  }
}
```
新的依赖`"angular-resource": "1.4.x"`告诉bower去安装一个和版本1.4.x兼容的angular-resource。我们可以通过以下方式告诉bower去下载并安装该依赖。

```
npm install
```

```
警告： 如果在你运行npm install后发布了一个新版本的Angular，那么这个时候使用bower install去安装依赖可能会因为angular.js的版本之间的差异会有问题需要重装。如果你遇到这个问题，那么删除app/bower_components文件夹，再运行npm install。
```

```
注意： 如果全局装了bower，那么可以使用bower install，但是针对目前这个工程，我们已经预先配置了npm install去运行bower的命令。
```
---

### 模板
我们自定义的资源服务会定义在`app/js/services.js`，所以我们需要在layout模板中引入该文件。除此之外，我们也需要加载`angular-resource.js`文件，该文件包含了`ngResource`模块：
`app/index.html`
```
...
  <script src="bower_components/angular-resource/angular-resource.js"></script>
  <script src="js/services.js"></script>
...
```
---

### 服务
我们创建一个服务去提供访问服务器上手机数据的方法。
`app/js/services.js`
```
var phonecatServices = angular.module('phonecatServices', ['ngResource']);

phonecatServices.factory('Phone', ['$resource',
  function($resource){
    return $resource('phones/:phoneId.json', {}, {
      query: {method:'GET', params:{phoneId:'phones'}, isArray:true}
    });
  }]);
```
我们通过factory注册了一个服务“Phone”。factory函数和controller的构造函数类似，都能通过函数参数的方式宣明依赖。Phone服务宣明了一个`$resource`的依赖。

`$resource`服务使得可以通过几行代码轻松创建[RESTful](https://en.wikipedia.org/wiki/Representational_state_transfer)请求。在我们应用中，就可以不用更底层的[$http](https://docs.angularjs.org/api/ng/service/$http)服务了。

`app/js/app.js`
```
...
angular.module('phonecatApp', ['ngRoute', 'phonecatControllers','phonecatFilters', 'phonecatServices']).
...
```
我们需要将phonecatServices依赖添加到phonecatApp模块中。

---

### 控制器
我们对一些控制器进行了重构（`PhoneListCtrl` 和`PhoneDetailCtrl`），用新的`Phone`服务替换了底层的[$http](https://docs.angularjs.org/api/ng/service/$http)服务。在和通过RESTful暴露的数据来源进行交互的时候，Angular的`$resource`服务比`$http`更加容易使用，且更容易理解。

`app/js/controllers.js`.
```
var phonecatControllers = angular.module('phonecatControllers', []);

...

phonecatControllers.controller('PhoneListCtrl', ['$scope', 'Phone', function($scope, Phone) {
  $scope.phones = Phone.query();
  $scope.orderProp = 'age';
}]);

phonecatControllers.controller('PhoneDetailCtrl', ['$scope', '$routeParams', 'Phone', function($scope, $routeParams, Phone) {
  $scope.phone = Phone.get({phoneId: $routeParams.phoneId}, function(phone) {
    $scope.mainImageUrl = phone.images[0];
  });

  $scope.setImage = function(imageUrl) {
    $scope.mainImageUrl = imageUrl;
  }
}]);
```
在`PhoneListCtrl`里，我们将：
```
$http.get('phones/phones.json').success(function(data) {
  $scope.phones = data;
});
```
替换成
```
$scope.phones = Phone.query();
```

注意，当我们触发Phone服务的query方法时，我们没有传递任何回调函数。尽管它看起来结果同步返回了，但其实并不是这样。同步返回结果是'未来'，一个对象将在XHR返回结果后填充数据。由于Angular的数据绑定，我们能使用这个特性，并且绑定到我们的模板。然后，当数据到达时，视图会自动更新。

有时，依赖于将来对象，数据版本不能单独地完成我们想要的任何行为，所以在这些情况下，我们能添加一个回调去处理服务器响应。`PhoneDetailCtrl`控制器就使用了`mainImageUrl`回调。

---

### 测试

因为我们使用了[ngResource](https://docs.angularjs.org/api/ngResource)模块，我们有必要更新Karma的配置文件。

`test/karma.conf.js`:
```
files : [
  'app/bower_components/angular/angular.js',
  'app/bower_components/angular-route/angular-route.js',
  'app/bower_components/angular-resource/angular-resource.js',
  'app/bower_components/angular-mocks/angular-mocks.js',
  'app/js/**/*.js',
  'test/unit/**/*.js'
],
```
我们修改了单元测试去验证新加的服务使用了HTTP服务且按希望的处理它们。该测试也检测我们的控制器是否和服务正确的交互。

如果我们使用标准的`toEqual`方法，我们的测试会因为值和响应不匹配而失败。为了解决这个问题，我们使用了一个新定义的`toEqualData`方法[Jasmin matcher](http://jasmine.github.io/1.3/introduction.html#section-Matchers)。当`toEqualData`匹配两个对象时，它只会比较对象里的属性忽略里面的方法。

`test/unit/controllersSpec`:
```
describe('PhoneCat controllers', function() {

  beforeEach(function(){
    this.addMatchers({
      toEqualData: function(expected) {
        return angular.equals(this.actual, expected);
      }
    });
  });

  beforeEach(module('phonecatApp'));
  beforeEach(module('phonecatServices'));


  describe('PhoneListCtrl', function(){
    var scope, ctrl, $httpBackend;

    beforeEach(inject(function(_$httpBackend_, $rootScope, $controller) {
      $httpBackend = _$httpBackend_;
      $httpBackend.expectGET('phones/phones.json').
          respond([{name: 'Nexus S'}, {name: 'Motorola DROID'}]);

      scope = $rootScope.$new();
      ctrl = $controller('PhoneListCtrl', {$scope: scope});
    }));


    it('should create "phones" model with 2 phones fetched from xhr', function() {
      expect(scope.phones).toEqualData([]);
      $httpBackend.flush();

      expect(scope.phones).toEqualData(
          [{name: 'Nexus S'}, {name: 'Motorola DROID'}]);
    });


    it('should set the default value of orderProp model', function() {
      expect(scope.orderProp).toBe('age');
    });
  });


  describe('PhoneDetailCtrl', function(){
    var scope, $httpBackend, ctrl,
        xyzPhoneData = function() {
          return {
            name: 'phone xyz',
            images: ['image/url1.png', 'image/url2.png']
          }
        };


    beforeEach(inject(function(_$httpBackend_, $rootScope, $routeParams, $controller) {
      $httpBackend = _$httpBackend_;
      $httpBackend.expectGET('phones/xyz.json').respond(xyzPhoneData());

      $routeParams.phoneId = 'xyz';
      scope = $rootScope.$new();
      ctrl = $controller('PhoneDetailCtrl', {$scope: scope});
    }));


    it('should fetch phone detail', function() {
      expect(scope.phone).toEqualData({});
      $httpBackend.flush();

      expect(scope.phone).toEqualData(xyzPhoneData());
    });
  });
});
```
你应该能看到如下输出：
```
Chrome 22.0: Executed 5 of 5 SUCCESS (0.038 secs / 0.01 secs)
```

### 总结
现在我们知道了如何构建一个自定义的RESTful服务，我们可以到下一步去学习如何通过动画改进我们的应用。
