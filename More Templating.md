在这一步中，你将实现phone details界面，这个页面会在用户点击手机列表中某一个手机时显现。

* 当你点击列表中某一个手机时，手机的详细信息会显示出来

为了实现phone details界面，我们将会使用$http去获取我们的数据，并且填充phone-detail.html视图模板。

重要的改变如下所示。你可以通过[GitHub](https://github.com/angular/angular-phonecat/compare/step-7...step-8)查看差异

---

### 数据

除了phones.json之外，app/phones/目录下也包含了每个手机详细信息的JSON文件：

app/phones/nexus-s.json

```
{
  "additionalFeatures": "Contour Display, Near Field Communications (NFC),...",
  "android": {
      "os": "Android 2.3",
      "ui": "Android"
  },
  ...
  "images": [
      "img/phones/nexus-s.0.jpg",
      "img/phones/nexus-s.1.jpg",
      "img/phones/nexus-s.2.jpg",
      "img/phones/nexus-s.3.jpg"
  ],
  "storage": {
      "flash": "16384MB",
      "ram": "512MB"
  }
}
```
每个这样的文件用同样的数据结构保存了手机的各种属性信息。我们将会在phone details视图里展示这些信息。

---

### 控制器

我们将通过$http服务去获取JSON文件来扩展PhoneDetailCtrl。工作方式类似于phone list controller。

`app/js/controllers.js`:

```
var phonecatControllers = angular.module('phonecatControllers',[]);

phonecatControllers.controller('PhoneDetailCtrl', ['$scope', '$routeParams', '$http',
  function($scope, $routeParams, $http) {
    $http.get('phones/' + $routeParams.phoneId + '.json').success(function(data) {
      $scope.phone = data;
    });
  }]);
```

为了构造HTTP请求的URL，我们通过使用$route服务，从当前的route中提取了`$routeParams.phoneId`

---

### 模板

TBD的占位符被手机的详细信息替换了。注意我们通过使用Angular的`{{expression}}`标记和`ngRepeat`将手机数据从模型映射到视图。

`app/partials/phone-detail.html`:

```
<img ng-src="{{phone.images[0]}}" class="phone">

<h1>{{phone.name}}</h1>

<p>{{phone.description}}</p>

<ul class="phone-thumbs">
  <li ng-repeat="img in phone.images">
    <img ng-src="{{img}}">
  </li>
</ul>

<ul class="specs">
  <li>
    <span>Availability and Networks</span>
    <dl>
      <dt>Availability</dt>
      <dd ng-repeat="availability in phone.availability">{{availability}}</dd>
    </dl>
  </li>
    ...
  <li>
    <span>Additional Features</span>
    <dd>{{phone.additionalFeatures}}</dd>
  </li>
</ul>
```

---

### 测试

我们写一个在步骤5为`PhoneListCtrl`写过的类似的单元测试。

`test/unit/controllersSpec.js`:

```
beforeEach(module('phonecatApp'));

  ...

  describe('PhoneDetailCtrl', function(){
    var scope, $httpBackend, ctrl;

    beforeEach(inject(function(_$httpBackend_, $rootScope, $routeParams, $controller) {
      $httpBackend = _$httpBackend_;
      $httpBackend.expectGET('phones/xyz.json').respond({name:'phone xyz'});

      $routeParams.phoneId = 'xyz';
      scope = $rootScope.$new();
      ctrl = $controller('PhoneDetailCtrl', {$scope: scope});
    }));


    it('should fetch phone detail', function() {
      expect(scope.phone).toBeUndefined();
      $httpBackend.flush();

      expect(scope.phone).toEqual({name:'phone xyz'});
    });
  });
...
```
你应该能看到如下输出：

```
Chrome 22.0: Executed 3 of 3 SUCCESS (0.039 secs / 0.012 secs)
```
我们增加了一个新的端到端测试，这个测试导航到Nexus S细节页面并且验证页面的标题是"Nexus S"。

`test/e2e/scenarios.js`:

```
...
  describe('Phone detail view', function() {

    beforeEach(function() {
      browser.get('app/index.html#/phones/nexus-s');
    });


    it('should display nexus-s page', function() {
      expect(element(by.binding('phone.name')).getText()).toBe('Nexus S');
    });
  });
...
```
你现在可以重新运行`npm run protractor`查看测试运行。

---

### 实验

使用[Protractor API](http://angular.github.io/protractor/#/api)，写一个测试去验证我们在Nexus S details页面显示了四个缩略图。

---

### 总结

现在我们已经有了手机详细页面，到第九步去学习怎么写自定义的显示过滤器。
