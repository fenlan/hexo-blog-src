---
title: laravel学习之旅
tag: laravel
date: 2017-07-30 15:18:23
categories: php
---

## 路由
**1.基本路由**
``` php
Route::get('basic1', function () {
    return 'Hello World';
});
```

**2.路由参数**
``` php
Route::get('user/{id}', function ($id) {
    return 'User-id-' . $id;
})->where('id', '[0-9]+');

Route::get('user/{name?}', function ($name = 'fenlan') {
    return 'User-name-' . $name;
})->where('name', '[A-Za-z]+');
```
<!--more-->
**3.路由群组**
``` php
Route::group(['prefix' => 'member'], function () {
    Route::get('user/member-center', ['as' => 'center', function () {
        return route('center');
    }]);
    Route::any('multy1', function () {
        return 'member-multy1';
    });
});
```

**4.控制器关联**
``` php
Route::get('member/{id}', 'MemberController@info');
```
> get参数中第一个是url路由，第二个是控制器MemberController及控制器中函数info

## 控制器
``` php
<?php

namespace App\Http\Controllers;

class MemberController extends Controller
{
    public function info($id) {
        //return 'member-info-id-' . $id;
        //return view('welcome');
        return view('info', [
            'name' => 'fenlan',
            'age' => 18,
            'sex' => 'male'
        ]);
    }
}
```

## 视图
``` php
info blade php
{{$name}}
{{$age}}
{{$sex}}
```
**laravel新建视图完成后需要模块编译(我是这么理解的)**
``` php
php artisan serve
```

## 连接数据库
**在.env文件中修改配置**
``` bash
APP_NAME=Laravel
APP_ENV=local
APP_KEY=
APP_DEBUG=true
APP_LOG_LEVEL=debug
APP_URL=http://localhost

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=homestead
DB_USERNAME=homestead
DB_PASSWORD=secret

BROADCAST_DRIVER=log
CACHE_DRIVER=file
SESSION_DRIVER=file
QUEUE_DRIVER=sync

REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379

MAIL_DRIVER=smtp
MAIL_HOST=smtp.mailtrap.io
MAIL_PORT=2525
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null

PUSHER_APP_ID=
PUSHER_APP_KEY=
PUSHER_APP_SECRET=
```
**修改完成后再编译一下，就连接成功**

## 数据库操作
**1.使用DB facade实现CURD**
``` php
<?php

namespace App\Http\Controllers;

use Illuminate\Support\Facades\DB;

class StudentController extends Controller
{
    public function test1() {
        $student = DB::select('SELECT * FROM student');
        dd($student);
        // $bool = DB::insert('INSERT INTO student(name, ID) VALUES(?, ?)', ['fenlan', '15130110067']);
        // $num = DB::update('UPDATE student SET in_time = ? WHERE ID = ?', ['2015', '15130110067']);
        // $num = DB::delete('DELETE FROM student WHERE id > ?', ['15130110067']);
    }
}
```
**这种方法属于原始方法，接下来有更酷的操作数据库方法**

**2.查询构造器及新增数据**
``` php
public function test2() {
        $bool = DB::table('student')->insert([
            ['ID' => '15130110098', 'name' => '小淳', 'sex' => 'female', 'class' => '1513011',
            'in_time' => '2015', 'status' => 'stay_in']
            ['ID' => '15130110099', 'name' => '小明', 'sex' => 'female', 'class' => '1513011',
            'in_time' => '2015', 'status' => 'stay_in']
        ]);
        var_dump($bool);

        $num = DB::table('student')->where('id', '15130110067')->update('in_time' => '2016');
        var_dump($num);
        $num DB::table('student')->increment('class', 1);
        var_dump($num);
        $num = DB::table('student')->where('id', '>=', '15130110067')->delete();
        var_dump($num);
        DB::table('student')->truncate();   // delete all data in table

        $students = DB::table('student')->get();     // return all data
        dd($students);
        $student = DB::table('student')->orderBy('id', 'desc')->first();
        dd($student);
        $names = DB::table('student')->pluck('name');    // return name attrubite
        dd($names);
        $names = DB::table('student')->lists('name', 'id');  // return id -> name(now this is removed)
        dd($names);
        $names = DB::table('student')->select('id', 'name', 'sex')->get();   // return attrubites
        dd($names);
        echo '<pre>';
        DB::table('student')->chunk(2, function($students) {
            var_dump($students);
        });     // for big data query, query 2 data every time
    }
```
**这个方法可以避免sql攻击注入，同时也人性化很多,还有很多内容，具体查看官方文档**
[database](https://laravel.com/docs/5.4/queries)

**3.Eloquent ORM**
``` php
<?php
// app\Student.php
namespace App;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * The table associated with the model.
     *
     * @var string
     */
    protected $table = 'student';
    protected $primaryKey = 'id';
}
```
``` php
// app\Http\Controllers\StudentController.php
<?php

namespace App\Http\Controllers;

use App\Student;

class StudentController extends Controller
{
    public function test() {
        $students = Student::all();
        $students = Student::find('15130110067')
        dd($students);

        $student = new Student();
        $student->ID = '15130110070';
        $student->name = '小青';
        $student->sex = 'female';
        $student->class = '1513011';
        $student->in_time = '2015';
        $student->status = 'stay_in';
        $bool = $student->save();
        dd($bool);
    }
}
```
``` php
// ORM 正则搜索
public function search(Request $request) {

    $keyword = $request->keyword;
    $keyword = '%' . $keyword . '%';

    // $books = DB::select('SELECT * FROM books WHERE keywords LIKE ?', [$keyword]);
    $books = Book::where('keywords', 'LIKE', $keyword)->paginate(12);
    if ($books->total()) {

        return view('book.searchResults', [
            'books' => $books,
        ]);
    } else {

        return view('book.searchNoFound');
    }
}
```
**具体内容查看官方文档**
[ORM](https://laravel.com/docs/5.4/eloquent)

## blade模板
``` php
// views\layout.blade.php
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>laravel模板继承</title>
    <style>
        .header {
            width: 1000px;
            height: 150px;
            margin: 0 auto;
            background: #f5f5f5;
            border: 1px solid #ddd;
        }

        .main {
            width: 1000px;
            height: 300px;
            margin: 0 auto;
            margin-top: 15px;
            clear: both;
        }

        .main .sidebar {
            float: left;
            width: 20%;
            height: inherit;
            background: #f5f5f5;
            border: 1px solid #ddd;
        }

        .main .content {
            float: right;
            width: 75%;
            height: inherit;
            background: #f5f5f5;
            border: 1px solid #ddd;
        }

        .footer {
            width: 1000px;
            height: 150px;
            margin: 0 auto;
            margin-top: 15px;
            background: #f5f5f5;
            border: 1px solid #ddd;
        }
    </style>
</head>
<body>
    <div class='header'>
        @section('header')
        头部
        @show
    </div>

    <div class='main'>
        <div class='sidebar'>
            @section('sidebar')
            侧边栏
            @show
        </div>
        <div class='content'>
            @yield('content', '主要内容')
        </div>
    </div>

    <div class="footer">
        @section('footer')
        尾部
        @show
    </div>

</body>
</html>
```
``` php
// views\student\test.php
@extends('layout')

@section('header')
    @parent
    header
@stop

@section('sidebar')
    sidebar
@stop

@section('content')
    content

    <!-- 1.模板中输出变量 -->
    <p>{{ $name }}</p>

    <!-- 2.模板中调用PHP代码 -->
    <p>{{ time() }}</p>
    <p>{{ date('Y-m-d H:i:s', time()) }}</p>

    <!-- 3.原样输出 -->
    <p>@{{ $name }}</p>

    {{-- 4.模板中注释 --}}

    {{-- 5.引入子视图 --}}
    @include('student.commen', ['message' => '我是错误信息'])

    @if ($name == 'fenlan')
        I'm fenlan
    @elseif ($name == 'hello')
        I'm hello
    @else
        Who am I?
    @endif

    <br>
    @unless ($name != 'fenlan')
        I'm fenlan
    @endunless

    <br>
    @for ($i=0; $i < 10; $i++)
        {{ $i }}
    @endfor

@stop
```

## Requests
``` php
<?php

namespace App\Http\Controllers;

use App\Student;
use Illuminate\Http\Request;

class StudentController extends Controller
{
    public function test(Request $request) {
        // 1.取值
        echo $request->input('name', 'default');
        echo $request->has('name');
        $res = $request->all();
        dd($res);

        // 2.判断请求类型
        echo $request->method();
        echo $request->isMethod('GET');
        $res = $request->is('student/*');
        var_dump($res);
    }
}
```

## Session
``` php
// routes\web.php
Route::group(['middleware' => ['web']], function () {
   Route::any('session1', 'StudentController@session1');
   Route::any('session2', 'StudentController@session2');
});
```
``` php
// 1.Http request session();
public function session1(Request $request) {
        $request->session()->put('key1', 'value1');
    }

    public function session2(Request $request) {
        echo $request->session()->get('key1');
    }
```
``` php
// 2.sesson()
public function session1(Request $request) {
        session()->put('key2', 'value2');
    }

    public function session2(Request $request) {
        echo session()->get('key2');
    }
```
``` php
// 3.Session class
    public function session1(Request $request) {
        Session::put('key3', 'value3');
    }

    public function session2(Request $request) {
        echo session::get('key3', 'default');
    }
```
``` php
public function session1(Request $request) {
        Session::push('student', 'fenlan');
        Session::push('student', 'shirk3');
    }

    public function session2(Request $request) {
        $res = Session::get('student', 'default');
        dd($res);
    }
```
``` php
public function session1(Request $request) {
        Session::forget('student');
        Session::flush('key', 'default');	// temporary
        Session::flush();
    }

    public function session2(Request $request) {
    	echo Session::has('student');		// return bool
        $res = Session::pull('student', 'default');
        dd($res);
    }
```

## Response
``` php
public function test() {
        // 响应json
        $data = [
          'errCode' => 0,
          'errMsg' => 'success',
          'data' => 'fenlan',
        ];

        return response()->json($data);
}
```
``` php
public function test() {
        // redirect
        return redirect('session2');
9}
```

## Middleware
**目的：实现一个功能，当访问时间小于活动开始时间，则跳转到宣传页面，当访问时间大于活动开始时间，则跳转到活动界面**
**1.宣传页面和活动页面构造**
``` php
// app\Http\Controller\StudentController.php
    // 宣传页面
    public function activity0() {

        return '活动快要开始啦，敬请关注';
    }

    // 活动页面
    public function activity1() {

        return '活动进行中，谢谢您的参与1';
    }

    // 活动页面
    public function activity2() {

        return '活动进行中，谢谢您的参与2';
    }
```
**2.添加Middleware**
``` php
// app\Http\Middleware\Activity.php
<?php

namespace App\Http\Middleware;

use Closure;

class Activity
{

    public function handle($request, Closure $next) {

        if (time() < strtotime('2017-08-03')) {

            return redirect('activity0');
        }

        return $next($request);
    }
}
```
``` php
// app\Http\Kernel.php 中添加一条Middleware
protected $routeMiddleware = [
        'auth' => \Illuminate\Auth\Middleware\Authenticate::class,
        'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
        'bindings' => \Illuminate\Routing\Middleware\SubstituteBindings::class,
        'can' => \Illuminate\Auth\Middleware\Authorize::class,
        'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
        'throttle' => \Illuminate\Routing\Middleware\ThrottleRequests::class,
        'activity' => \App\Http\Middleware\Activity::class,
    ];
```
**3.添加路由**
``` php
Route::group(['middleware' => ['activity']], function () {
    Route::any('activity1', 'StudentController@activity1');
    Route::any('activity2', 'StudentController@activity2');
});

Route::any('activity0', 'StudentController@activity0');
```
**4.浏览器访问activity1或者activity2**

## File
**项目中遇到一个问题，需要数据库储存图片，网上给出一个好的方式就是将图片的路径存进数据库，laravel中实现过程如下**
**1.提交图片表单**
``` php
<form class="form-horizontal" method="post" enctype="multipart/form-data" action="{{ url('test2') }}">
    {{ csrf_field() }}
<input type="file" name="photo">
<div class="form-group">
    <div class="col-sm-offset-2 col-sm-10">
        <button type="submit" class="btn btn-primary">提交</button>
    </div>
</div>
</form>
```
**2.显示图片页面**
``` php
<img height="150" width="150px" src="{{url('/images/'.$image)}}"/>
```
**3.控制器实现**
``` php
public function test2(Request $request) {

    $img = time() . '.' . $request->photo->getClientOriginalExtension();
    $path = $request->photo->move(public_path('images'),$img);

    return view('student.test2', [
        'image' => $img,
    ]);
}
```
**最后图片存进了laravel项目下的public/images**

## 踩坑总结
1. `TokenMismatchException in VerifyCsrfToken.php line 68`
**laravel 默认开启了 csrf验证 ，post请求需要验证csrf,所以要在表单里 加个隐藏域**
**解决方案：**
![](/images/laravel_problem_1.png)
2. `MassAssignmentException in Model.php line 232:`
**在添加学生的时候选择`action`为空时，会出现这个错误，解释为`new Student`时复制不能批量操作，因此要在`Model`中添加**
``` php
protected $fillable = ['name', 'age', 'sex'];
```
3. `ErrorException in HasAttributes.php line 403:
Relationship method must return an object of type Illuminate\Database\Eloquent\Relations\Relation`
**在写`Student`模型的时候，将一个方法命名为`sex`，但`Student`又有一个属性是`sex`，两者相冲突了，因此需要将`sex`方法重新命名以解决冲突**
4. `无法修改和删除数据`
**这是我遇到的巨坑的一次，原因在于`Student Model`中将主码写错了，原本应该是`id`，被我写成了`ID`，然后一直被找出来，之前就有预感是`Student Model`错了，但是我检查了很多遍都没注意，吐血。。。**
5. `PDOException in Connector.php line 55:could not find driver`
**就像报错说的没有找到`driver(驱动)`，所以少了什么呢，少了`php`连接`mysql`的`module(组件)`，组件名字`pdo_mysql`，安装组件后重启`php-fpm`和`nginx`**
``` bash
yum install php70w-mysql
```
6. `Laravel sessions not working on server`
**这个问题困扰我两天吧，然后先说遇到的问题，就是在浏览器中修改数据或者添加数据等操作后，应该在页面有一个提示消息，我单独分离出来用`Session`实现，在`php`内带服务器上可以正常工作，但是在我的`server`中却无法实现。解决方法是修改文件`model`**
``` bash
chmod -R a+rw storage/
```
7. `ErrorException in e4e354417ec7f106982a7198b9ad5688b9936b71.php line 1:
Undefined variable: errors`
**这个跟中间件有关系，主要实在5.2版本中会遇到，解决方案是将`Kernel.php`中的`$middlewareGroups`的内容移到`$middleware`中**
8. `No supported encrypter found. The cipher and / or key length are invalid.`
**执行`php artisan key:generate`就 ok**

## 项目源码
[laravel-ubuntu](https://github.com/fenlan/laravel-ubuntu)