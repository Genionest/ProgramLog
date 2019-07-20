1. 请求：

   1. ```php+HTML
      <?php
      // 引用
      use think\Request;
      
      $request = Request::instance();  // 创建一个Request对象
      $request = request();  // 这个是助手函数版本
      $request->domain();  // 获取当前域名
      $request->baseFile();  // 获取入口文件
      $request->ext();  // 获取当前的后缀（伪静态）
      $request->module();  // 获取当前模块
      $request->controller();  // 获取当前控制器
      $request->action();  // 当前操作
      $request->method();  // 获取当前请求方法
      ...
      // 变量获取
      $request->param('name');  // 获取当前请求的name变量
      $request->param();  // 获取当前请求的所有变量，自然是返回数组
      $request->get('name');  // 获取当前GET请求的name变量
      
      // 简单函数
      input('param.name');  // 获取当前请求的name变量
      input('name');  	 // 获取当前请求的name变量
      input('');  // 获取当前请求的所有变量
      input('get.name');  // 获取当前GET请求的name变量
      input('get.');  // 获取当前GET请求的所有变量
      
      // 强制转化类型
      $request->param('name/s');  // 获取到的结果强制转化为字符串
      input('name/s');  // 获取到的结果强制转化为字符串
      
      // 更改变量
      $request->get(['id'=>0]);  // 更改GET变量
      // param变量是无法通过这种方法修改的
      
      // 获取请求类型
      $request->isGet()  // 是否为GET情求
      
      
      // 这样就可以被系统识别为PUT请求
      
      // 伪静态
      // 在config.php里修改
      'url_html_suffix' => 'html',
      // 值为false就是关闭
      'url_html_suffix' => false,
      /* 使用伪静态之后，就会把 http://servername/Home/Blog/read/id/1 变成 http://servername/Home/Blog/read/id/1.html 
      */
      ?>
      <!-- 请求伪装 -->
      <form method='post'>
          <input type='hidden' name='_method' value='PUT'>
      </form>
      ```

2. 数据库

   1. 数据库配置，在application/database.php里进行配置，也可以使用 方法配置， 模型配置

   2. ```php
      // 原生查询
      // 首先要创建一个控制器
      use app\index\controller;
      use think\Db;
      
      class Index
      {
          public function index(){
              // 在方法里面操作
              // 查询用query(), 增删改用execute(), 里面输入sql语句
              $result = Db::execute('insert into think_data (name, status) values("Nep", 1)');  // 返回影响的行数
          }
      }
      ```

   3. ```php
      // 查询结构器
      // 插入
      Db::table('think_data')->insert(['name'=>"Nep", 'status'=>1]);
      // 更新，如果数据中包含主键，则无需用where
      Db::table('think_data')->where('id', 1)->update(['name'=>'Noc', 'status'=>1]);
      Db::table('think_data')->update(['id'=>1, 'name'=>"Noc", 'status'=>1]);
      // 查询
      Db::table('think_data')->where('status', 1)->select();
      // 单个查询（返回符合条件的第一个，返回这条数各字段组成的一维数组）
      Db::table('think_data')->where('status', 1)->find();
      // 删除
      Db::table('think_data')->where('id', 1)->delete();
      
      // 不带前缀的写法，如果配置中定义了前缀'think_',
      Db::name('data')->insert(['name'=>"Nep", 'status'=>1]);
      Db::name('data')->where('id', 1)->update(['name'=>'Noc']);
      
      // 使用助手函数，也是不带前缀的，这样每次都会重连数据库，不推荐使用
      $db = db('data');
      $db->insert(['name'=>"Nep", 'status'=>1]);
      
      // 链式操作，可以完成的操作
      // 链式操作不分先后，但要在关键方法之间（比如select）
      $list = Db::name('daat')
          ->where('status', 1)
          ->field('id,name')
          ->order('id', 'desc')
          ->limit(10)
          ->select();
      
      // 查询表达式
      // where('字段名', '条件表达式', 查询值)
      /* 条件表达式有 =(默认), neq, >, >=, like, between, ... */
      $result = Db::name('data')->where('name', 'like', '%e%')->select();  // 模糊查询
      $result = Db::name('data')->where('id', 'between', [2, 6])->select();  // 查询id为2-6之间的数据
      
      // 添加多条数据
      $data = [
          ['name'=>"Nep", 'status'=>1],
          ['name'=>"Noc", 'status'=>1],
      ];
      Db::name('data')->insertAll();
      
      // 更新某个字段的值，返回影响的行数，这样做不用定义所有字段的值
      Db::name('data')->where('id', 1)->setField('name', "Noc");
      // 自增/自减一个字段的值，其他跟上面差不多
      Db::name('data')->where('id', 1)->setInc('status');  // 'status'字段自增1
      Db::name('data')->where('id', 1)->setInc('status', 2);  // 'status'字段自增2
      Db::name('data')->where('id', 1)->setDec('status');  // 'status'字段自减1
      ```

3. 模型：

   1. ORM：表->记录，记录->对象，字段->对象属性

   2. ```php
      // 定义模型
      namespace app\index\model;
      
      use think\Model;
      /*
      每个模型自动对应一个数据表, 命名和数据库相联系，定义了前缀就不带前缀，大写驼峰
      假设数据库名为think_user, 前缀定义为think_
      */
      class Data extends Model  
      {
          
      }
      
      // 业务逻辑由控制器处理，所有我们在控制器里写
      namespace app\index\controller;
      use app\index\model\User as UserModel;  // 为了不与下面的User重名
      class User
      {
          // 新增数据
          public function add(){
              // 方法1
              $user = new UserModel();
              $user->name = "Nep";
              $user->status = 1;
              if($user->save()){  // insert和update都是save方法
                  return '用户新增成功';
              }else{
                  return '用户新增失败';
              }
              // 方法2
              $user['name'] = "Nep";
              $user['status'] = 1;
              if($result=UserModel::create($user)){
                  return '用户新增成功';
              }else{
                  return '用户新增失败';
              }
          }
          // 批量新增
          public function addList(){
              $user = new UserModel();
              $list = [
                  ['name'=>"Nep", 'status'=>1],
                  ['name'=>"Noc", 'status'=>1],
              ];
              if($user->saveAll($list)){
                  return '批量插入成功';
              }else{
                  return '批量插入失败';
              }
          }
          // 更新数据
          public function update(){
              // 方法1
              $user = UserModel::get(1);  // 与新增不同的就是先获取
              $user->name = "Noc";
              $user->status = 1;
              if($user->save()){
                  return '数据跟新成功';
              }else{
                  return '数据新增失败';
              }
              // 方法2
              $user = new UserModel();
              $user->save(['name'=>"Noc", 'status'=>1], ['id'=>1]);
          }
          // 批量更新
          public function updateAll(){
              $user = new UserModel();
              $list = [
                  ['id'=>1, 'name'=>'Noc', 'status'=>0],
                  ['id'=>2, 'name'=>'Bla', 'status'=>0],
              ];
              $user->saveAll($list);
          }
          // 查询数据
          public function select(){
              // 方法1
              $user = UserModel::get(1);
              echo $user->name;
              // 方法2
              $user = UserModel::get(['name'=>'Nep']);
              echo $user->status;
              // 方法3
              $user = UserModel();
              $result = $user->where('id', 1)->find();
              echo $result->name;
              // get和find方法返回的都是当前模型的实例，可以使用模型的方法
              
              // 批量查询
              // 方法1
              $list = UserModel::all('1, 2, 3');
              foreach($list as $key=>$value){
                  echo $value->name;
              }
              // 方法2
              $list = UserModel::all([1, 2, 3]);
              foreach($list as $key=>$value){
                  echo $value->name;
              }
              // 方法3
              $user = new UserModel();
              $result = $user->where('status', 1)->limit(2)->select();
              foreach($result as $key=>$value){
                  echo $value['name'];  // 也可以这样获取属性值
              }
              // all和select方法返回的都是二维数组
              
              // 聚合
              UserModel::count();  // 所有数据数量
              UserModel::where('status', '>', 0)->count();  // 统计数量
              UserModel::where('status', 1)->avg('id');  // 获取平均值 
              UserModel::max('id');  // 获取最大值
          }	
          public function delete(){
              // 方法1
              $user = UserModel()::get(1);
              if($user->delete()){
                  return '数据删除成功';
              }else{
                  return '数据删除失败';
              }
              // 方法2，根据主键删除
              if(UserModel::destroy(2)){
                  return '数据删除成功';
              }else{
                  return '数据删除失败';
              }
              // 批量删除，就不写return了
              UserModel::destroy([1, 2, 3]);
              UserModel::destroy('1, 2, 3');
              $result = UserModel::where('status', 1)->delete();
              // 条件删除
              UserModel::destroy(['status'=>1]);
          }
      }
      // 预防注入式攻击
      $user->allowField(true)->save($data);  // 更新
      ```

4. 视图：

   1. ```php+HTML
      <?php
      // 模板赋值
      namespace index\app\controller;  // 命名空间看文件位置
      class Index extends Controller
      {
          public function index(){
              // 模板变量赋值
              $name = 'Nep';
              $status = 1;
              $this->assign('name', $name);
              $this->assign('status', $status);
              // 或者批量赋值
              $this->assign([
                  'name'=>"Nep",
                  'status'=>1,
              ]);
              // 模板输出
              // \think\View类的fetch方法，继承了Controller就有了
              // fetch('模板文件', [模板变量(数组)])
              // 模板文件自动定位：当前模块/view/当前控制器(小写)/当前方法(小写).后缀
              return $this->fetch('index', [
                  "name"=>'Nep',  // fetch和assign传变量作用是一样的
                  'status'=>1,
              ]);
              
              // 不使用fetch，也可以使用助手函数
              return view('index', [变量懒得写了]);
          }
      }
      ?>
      <!-- 在view的模板文件中用{$name}和{$status}就可以获取这两个变量的值了 -->
      <div>{$user}</div>
      <div>{$status}</div>
      ```

   2. 

      ```php
      // 模板标签
      // 1. 普通标签，用于变量获取和注释，形如：{$name}，中间不能有空格
      // 可以在application\config.php 里的修改
      'tpl_begin' => '{',
      'tpl_end' => '}',
      {literal}'里面的标签不解析'{/literal}
      // {// 注释内容} {/*多行注释*/} 中间不能有空格
      // 2. 标签库标签
      
      // 3. 系统变量
      // 一般以$Think打头
      {$Think.server.server_name}  // 输出$_SERVER['SERVER_NAME']
      {$Think.session.user_id}  // 输出$_SESSION['user_id']
      {$Think.get.pageNumber}  // 输出$_GET['pageNumber']
      {$Think.cookie.name}  // 输出$_COOKIE['name']
      // 常量输出
      // 在controller里
      define('APP', '常量值');
      // 在view里
      {$Thik.APP}  // 输出'常量值'
      ```

   3. ```php+HTML
      <?php
      // 模板布局
      // 1. 全局配置，使用于 所有网页头尾都一样 的情况，
      // 在public/config.php里
      'template' => [
          'layout_on' => true,
          'layout_name' => 'layout',
          'layout_item' => '{__CONTENT__}',  // 你可以改成{__REPLACE__}或者其他
      ],
      ?>
      <!-- 没开启layout，是渲染index/view/index/index.html文件 -->
      <!-- 开启layout后，是渲染index/view/index/layout.html，而且其中的{__CONTENT__}部分会由index/view/index/index.html替代 -->
      <!-- layout.html -->
      <div></div>
      {__CONTENT__}  <!-- 这一部分会替换成index.html的内容 -->
      <div></div>
      <?php
      {__NOLAYOUT__}  // 在配置开启layout的情况下，让你不使用layout
      // 2. 模板标签方式
      // 配置里没有开启layout的情况下，可以在view/index/index.html里使用这个标签
      {layout name='layout' /}  // name=模板路由
      ?>
      ```

   4. ```php+HTML
      <?php
      // 模板继承，是更加灵活的模板布局
      // 在 index/view/index/base.html 里
      {block name="title"}模块{/block}
      /* 这种方法相当于 一个类方法，在其他 类(视图文件)中可以继承，然后就默认拥有了这些类方法
      */
      // 在index/view/index/index.html 里
      {extend name="base" /}
      /* 除非你在下面又写了(重载)一个{block name="title"}{/block}，不然base中的这一部分会被继承下来
      */
      // 如果在block标签里使用{__block__}，那么就不会被重载，而是合并新的内容，并且用了之后block标签外的内容都无效了
      ?>
      
      <?php
      // 包含文件
      // include标签会被引用文件的内容替代
      {include file="模板文件1，模板文件2，..." /}
      // 包含的模板文件中不能使用 布局 或 模板继承
      // 路径是以项目目录/public/路径下为起点
      {include file='../application/index/view/index/lang.html' /}
      // 包含文件中可以使用include标签，但不能是A包含A，或A，B互相包含 这种死循环
      ?>
      ```

   5. 

      ```php+HTML
      // 对模板输出变量使用函数
      {$data.name|md5} 
      <?php echo (md5($data.name)); ?>  // 编译后
      
      {$create_time|date="y-m-d", ###}  // ###类似于前面$create_time的占位符
      <?php echo (date("y-m-d", $create_time)); ?>
      
      {$name|md5|strtoupper|substr=0,3}
      <?php echo (substr(strtoupper(md5($name)), 0, 3)); ?>
      // 简单的写法
      {:substr(strtoupper(md5($name)), 0, 3)}  // 能理解{:url()}了
      
      <?php echo (substr(strtoupper(md5($name)), 0, 3)); ?>
      ```

   6. ```html
      <!-- 内置标签 -->
      {volist name='list' id='vo' offset='0' length='10'}
      <!-- name获取$list的值 id是循环变量$vo offset设置起始元素 -->
      {/volist}
      
      {for start='开始值' end='结束值' comparison='' step='步长' name='循环变量名'}
      {/for}
      
      {php}php原生语句;{/php}  <!--里面用不了内置标签-->
      ```

   7. 

5. 路由：

   1. \think\Route类

   2. 默认path_info规则：`http://server/module/controller/action/param/value/...`

      `http://服务器/模块/控制器/方法/参数/值/...`

   3. 模式：(config.php中配置)

      * 普通模式`'url_route_on'=>false, 'url_route_must'=>false,` 

      * 混合模式`'url_route_on'=>true, 'url_route_must'=>false,` 只对定义路由的 访问地址 定义路由规则

      * 强制模式`'url_route_on'=>true, 'url_route_must'=>true,` 所有访问地址 必须定义路由规则（包括首页）

      * ```php+HTML
        // 在route.php文件中写，不是在其他文件中
        use think\Route;
        
        Route::get('/', function(){  // 首页用'/'就可以了
        	return 'Hello, world!';
        });
        ```

      * ```php
        /* 动态注册
        Route::rule('路由表达式', '路由地址', '请求类型', '路由参数(数组)', '变量规则(数组)');
        路由地址 模块/控制器/函数，模块就是index或者admin，模块下有mvc，控制器 就是模块下的controller文件夹里的 php文件
        */
        // 还是在route.php里
        use think\Route
        
        Route::rule('new/:id', 'index/Index/list')；
        /* 访问http://serverName/new/5
        就会访问到http://servername/index/index/list/id/5 
        :id 带冒号代表输入一个参数，不是说要写上这个冒号
        */
        // 定义了请求类型，则只有符合时才能生效
        // 请求类型有"GET","POST","PUT","DELETE",*(表示所有，默认)
        // 如果同时要GET和POST,可以写"GET|POST"
        // 也有简单的方法
        Route::get('new/:id', 'News/read');  // 定义GET请求路由规则
        Route::any('new/:id', 'News/read');  // 支持任意请求路由规则
        // 批量注册
        Roue::get(['new/:id'=>'News/read', 'blog/:name'=>'Blog/detail']);
        ```

      * ```php
        // 在route.php的return的数组里直接添加 也可以配置路由
        return [
        	'new/:id' => 'News/read',
        	'blog/:id'=> ['Blog/update', ['method'=>'post|put'], ['id'=>'\d+']],
        ]
        ```

      * 定义路由配置文件：`'route_config_file'=>['route', 'route2', 'route3'],`

      * ```php
        /*资源路由，还是在route.php里
        使用Route::resource('路由表达式', '路由地址');
        路由地址: 模块/控制器
        */
        Route::resource('blog', 'index/blog');
        // 或者在return使用'__rest__'
        return [
        	'__rest__' => [
        		// 指向index模块的blog控制器
        		// 指向index/controller/Blog.php
        		'blog' => 'index/blog',
        	],
        ]
        /*
        设置之后会自动注册7个路由规则
        [标识,  请求类型 路由规则  		函数名]
        [index,	 GET,	blog,			index]
        [create, GET,	blog/create,	 create]
        [save, 	 POST,	blog,			save]
        [read,	 GET,	blog/:id		read]
        [edit,	 GET,	blog/:id/edit,	 edit]
        [update, PUT,	blog/:id,		update]
        [delete, DELETE, blog/:id,		 delete]
        你只需要在你的控制器里 index/controller/Blog.php里
        写好你需要的几个函数就行了(函数名和路由在上面)
        */
        ```

      * ```php
        /*快捷路由
        让你通过定义函数名 就可以定义路由和请求类型
        现在不能通过return的数组实现了
        Route::controller('路由表达式', '路由地址');
        路由地址：模块\控制器
        */
        
        // 在route.php里
        Route::controller('user', 'index/User')  \\ 路由不区分大小写的
        
        // 在index\controller\User.php里
        namespace app\index\controller;
        class User
        {
        	public function getInfo(){
        		// get请求访问 http://servername/user/info 调用此方法
        	}
        	public function postInfo(){
        		// post请求访问 http://servername/user/info 调用此方法
        	}
        	public function getPhone(){
        		// get请求访问 http://servername/user/phone 调用此方法
        	}
        }
        ```

6. 自动完成

   1. ```php
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

   2. ```php
      // 数据自动完成
      /* 可以使用auto, insert, update三个属性，分别在写入、新增、更新时 在数据库中自动完成字段。auto包括新增和更新
      */
      namespace app\index\model;
      use think\Model;
      
      class User extends Model
      {
          // 新增或保存时会自动完成'name', 'ip', 'password'
          protected $auto = ['name', 'ip', 'password'];  // 定义
          // 新增时会自动完成'status'
          protected $insert = ['status'=>1];
          // 保存时会自动完成'ip'
          protected $update = ['ip'];
          // 按以下格式定义函数
          protected function setNameAttr($value){
              return strtolower($value);
          }
          protected function setIpAttr(){
              return request()->ip();
          }
          protected function setPasswordAttr($value){
              return md5($value);
          }
      }
      
      // 在新增数据时，会对name, ip, status 字段自动完成
      $user = new User;
      $user->name = 'thinkphp';
      $user->save();  // 新增时
      
      $user = User::find(1);
      $user->name = 'THINKPHP';
      $user->save();  // 保存时
      
      ```

   3. ```php
      // 验证器
      // 创建验证器类
      // \app\index\validate\User下
      namespace app\index\validate;
      
      class User extends Validate
      {
          // 定义规则
          protected $rule = [
              'name' => 'require|max:25',  // require代表不能为空
              'email' => 'email',
              'password|密码' => 'require|min:3|confirm:repassword',  // confirm:repassword代表repassword要和password一致
          ];
          // 定义验证未通过是的提示信息
          protected $message = [
              // 每个验证对应一个
              'name.require' => '用户名不能为空',
              'name.max' => '用户名的长度不能超过25',
              'password.require' => '密码不能为空',
              'password.min' => '密码长度不能小于3',
              'password.confirm' => '两次输入不一致',
              'email.email' => '邮箱格式错误',
          ];
      }
      
      // 验证时，在控制器里
      $data = input('');
      $val = new UserValidate();
      if(!$val->check($data)){
          $this->error($val->getError());
          exit;03
              
      }
      ```

7. 跳转

   1. ```php
      $this->error('信息', '跳转路径');  // 默认跳转回上一页
      $this->success('信息', '跳转路径');
      ```

8. 分页

   1. ```php
      // 方法1
      // 在 controller
      $list = User::where('status', 1)->paginate(10);
      // 把分页数据赋值给模板变量list
      $this->assign('list', $list);
      // 渲染
      $this->fetch();
      // 在模板文件里 view
      {$list->render()}  // 对参数进行render
      
      // 方法2
      // 在 controller
      $this = User::where('statuts', 1)->paginate(10);
      // 获取分页显示
      $page = $list->render();  // 对参数进行渲染，具体看是什么，不是一定写$list
      // 在模板变量中赋值 
      $this->assign('list', $list);
      $this->assign('page', $page);
      // 渲染模板输出
      return $this->fetch();
      ```

9. 验证码

   1. ```php
      
      ```

   2. 

