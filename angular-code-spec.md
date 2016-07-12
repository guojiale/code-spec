#Angular 最佳实践
---
参考：

* [angular-styleguide](https://github.com/johnpapa/angular-styleguide/blob/master/a1/README.md)
* [angularjs-best-practices-directory-structure](https://scotch.io/tutorials/angularjs-best-practices-directory-structure)
* 
---

## <a name="contents">目录</a>

1. [项目结构](#project)
1. [最佳实践、规范](#best)
1. [优化](#optimize)

## <a name="name">项目结构</a>
- 标准结构(适用小型项目)
```javascript
app/
----- controllers/
---------- mainController.js
---------- otherController.js
----- directives/
---------- mainDirective.js
---------- otherDirective.js
----- services/
---------- userService.js
---------- itemService.js
----- js/
---------- bootstrap.js
---------- jquery.js
----- app.js
views/
----- mainView.html
----- otherView.html
----- index.html    
```
这是比较经典的项目结构，以类型分类，好处就是一看目录就知道里面是什么文件，当文件不多的时候也是容易查找阅读。但是，当每一个分类的文件都比较多的时候，不仅混乱难查找，也更难维护，理解修改成本较大。

- 更好的结构(适用中大型项目)
```javascript
app/
----- shared/   // acts as reusable components or partials of our site
---------- sidebar/
--------------- sidebarDirective.js
--------------- sidebarView.html
---------- article/
--------------- articleDirective.js
--------------- articleView.html
----- components/   // each component is treated as a mini Angular app
---------- home/
--------------- homeController.js
--------------- homeService.js
--------------- homeView.html
---------- blog/
--------------- blogController.js
--------------- blogService.js
--------------- blogView.html
----- app.module.js
----- app.routes.js
assets/
----- img/      // Images and icons for your app
----- css/      // All styles and style related files (SCSS or LESS files)
----- js/       // JavaScript files written for your app that are not for angular
----- libs/     // Third-party libraries such as jQuery, Moment, Underscore, etc.
index.html
```
这是以模块划分，组件化的概念的项目结构。同一个功能模块的相关文件都在一个文件夹内，一幕了然，职责清晰。这种结构可能对新手来说难理解一点，但随着对项目的熟悉，这种结构也会带来更大的工作效率。

## <a name="beset">最佳实践、规范</a>
请阅读：[Angular规范](https://github.com/johnpapa/angular-styleguide/blob/master/a1/i18n/zh-CN.md#%E6%89%8B%E5%8A%A8%E4%BE%9D%E8%B5%96%E6%B3%A8%E5%85%A5)
## <a name="optimize">优化</a>
- ### 减少监控值的个数  
    由于在html模板中绑定的数据，angular都会为其创建相应的监听器，绑定的数据越多，监听器越多，那么angular在进行脏值检测的时候遍历的监听器也就越多，性能就会降低。所以当我们的页面渲染完成后不需要更新。则采用单次绑定，仅在初始化的时候绑定：
    单个值：
    ```javascript
    <div>{{::value}}</div>
    ```
    列表：
    ```javascript
    <div ng-repeat="item in ::list"></div>
    ```

- ### 使用track by 提升索引的性能
    在angular中，在数据变更时DOM也会相应的更新，所以，DOM和数据需要建立关联，即需要一个映射关系，映射关系需要唯一的索引，angular默认对简单类型使用自身当索引，当出现重复的时候，就会出错了。如果指定$index，也就是元素在数组中的下标为索引，就可以避免这个问题。
    简单数组：
    ```javascript
    this.arr = [1, 3, 5, 3];
    ```
    ```javascript
    <!-- 报错 -->
    <ul>
        <li ng-repeat="num in arr">{{num}}</li>
    </ul
    ```
    ```javascript
    <!-- 指定$index为索引 -->
    <ul>
        <li ng-repeat="num in arr track by $index">{{num}}</li>
    </ul
    ```

    对象数组：
    ```javascript
    this.arr = [{
        id: 1,
        name: 'naraku666'
    }, {
        id: 2,
        name: 'ljs'
    }, ...];
    ```
    ```javascript
    <!-- 性能较差 -->
    <ul>
        <li ng-repeat="item in arr">{{item}}</li>
    </ul
    ```
    ```javascript
    <!-- 指定唯一值id为索引 -->
    <ul>
        <li ng-repeat="item in arr track by item.id">{{item.name}}</li>
    </ul
    ```

- ### 使用`ng-cloak`指令在angular启动完成之前隐藏不需要显示的html,避免闪烁。  

    ```html
     <div id="template1" ng-cloak>hello</div>
    ```

- ### 离开页面时注销不需要的监听器、定时器和事件绑定。
    因为这些操作可能会在页面跳转后继续执行，影响性能
    
    ```javascript
    var destroyNameWatch = $scope.$watch('user.name', function(){
        ....
    });
    var timer = $interval(function(){
        ....
    });

    // 监听页面离开事件
    $scope.$on('$destroy', function(){
        // 注销监听器
        destroyNameWatch();
        // 清除定时器
        $interval(timer);
    });
    ```