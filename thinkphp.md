1. mvc

   * controller/Ab.php fa()  对应  view/Ab/fa.html
   * model/TpBbs.php  对应  数据库里的 tp_bbs表

2. controller里用 `$this->assign('argu', $argu);` 传参，view里面通过php或内置标签获取这些参数；controller里最后 `return $this->fetch();` 渲染出来

3. 内置标签 `{volist name='$argu' id='i'}{\volist}`

4. 分页：

   1. 在controller里 数据库查询后面加入paginate() `$this=Db::all()->paginate(5)`，在view里加入`{$list->render()}` 即可实现简单分页

   2. ```php+HTML
      // 通过设置paginate的参数
      ->paginate(2, false, [
      	'type' => 'Bootstrap',  \\ 分页视图渲染路径
      	'var_page' => 'page',
      	'query' => ['a'=>1, 'b'=>'b', 'c'=>input('param.page')],  \\ 传入更多参数
      ])
      ```

   3. 修改配置文件，使用扩展：首先在extend\page目录下新建一个文件，假设是Page.php；然后再到application/config.php里修改 分页配置paginate的type参数，改为 `'type'=>'page\Page'`

5. 表单提交：

   1. ```php+HTML
      // view里面
      <form method='post' action={:url(...)}>
          
      <form>
          
      // controller里面
      <?php
          $request = Request::instance();  // 实例化一个Request类
          if($request->method()=="POST"){  // 判断是否为POST方法提交
              $name = input['name'];  // 获取当前请求的name参数
          }
          // 实例化一个模型
          $user = new M;
          $user->name = $name;  // 数据库的name字段
          $user->save();
      ```

   2. 

6. 时间戳：

   1. ```php+HTML
      /*
      时间戳默认是create_time, update_time这两个字段，类型为int
      */
      
      // 配置里开启
      // 路径：application/datebase.php
      auto_timestamp => true;
      
      // 单个模型中开启
      protected $autoWriteTimestamp = true;
      
      // 在模型中设置时间戳字段名
      // 默认为create_time, update_time这两个
      protected $createTime = 'new_create_time';
      protected $updateTime = 'new_update_time';
      ```

7. 视图模板组合

   1. 使用`{include file='...' /}`

8. 管理员模块：其实application/index就是一个模块，你可以复制一个它改名admin，也就实现了管理员模块的创建，浏览器里 url 为 public/admin/..

9. `{:url('route/func', ['argu0'=>$argu0, 'argu1'=>$argu1])}`这货还能传参，不过也是，第一个参数就是函数呀。

10. * 跳转：`$this->success('文字', 'index/index');$this->error('文字', 'index/index');`
    * 重定向：`$this->redirect('http://www.wargon.com/index');` 

11. ```php
    // 软删除
    // 首先需要引入
    namespace app\index\model;
    
    use think\Model;
    use traits\model\SoftDelete;
    
    class User extends Model
    {
        use SoftDelete;
        $protected $deleteTime = 'delete_time';  // 定义软删除字段
        
    }
    
    // 定义好了之后，就可以在控制器里使用了
    User::destroy(1);  // 软删除
    User::detstroy(1, true);  // 真实删除
    // 或者
    $user = User::get(1);
    $user->delete();  // 软删除
    $user->delete(true);  // 真实删除
    // 默认情况下查询数据不包含软删除的数据，如果你一定要，可以
    User::withTrashed()->find();
    User::withTrashed()->select();
    // 如果你只要查询软删除的数据
    User::onlyTrashed()->find();
    User::onlyTrashed()->select();
    ```

12. 

