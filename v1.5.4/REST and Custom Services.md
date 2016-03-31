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
我们创建一个服务区提供访问服务器上手机数据的方法。
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
在`PhoneList`里，我们将：
```
$http.get('phones/phones.json').success(function(data) {
  $scope.phones = data;
});
```
替换成
```
$scope.phones = Phone.query();
```
