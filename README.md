# tp5_import_excel
tp5导入excel

> https://my.oschina.net/laobia/blog/1633943

首先:在extend里面引入PHPexcel文件,直接根目录导入进去
 
html创建上传按钮

```html
<form  class="layui-form" method="post" action="">
    <div class="layui-form-item" >
        <label class="layui-form-label">上传excel</label>
        <div class="layui-input-inline">
            <div class="layui-upload">
                <button type="button" name="myfile" class="layui-btn" id="myfile"><i class="layui-icon"></i>上传文件</button>
            </div>
        </div>
    </div>
    <div class="layui-form-item" style="padding-left: 35%;">
        <div class="layui-input-inline"  >
            <button class="layui-btn" lay-submit lay-filter="formsub">立即提交</button>
            <button type="reset" class="layui-btn layui-btn-primary">重置</button>
        </div>
    </div>
</form>
```

```js
<script type="text/javascript">
    layui.use(['form','upload'],function(){
        var form=layui.form;
        var upload=layui.upload;
        upload.render({ //允许上传的文件后缀
            elem: '#myfile'
            ,url: "{:url('sale/do_upload')}"
            ,accept: 'file' //普通文件
            ,exts: 'xls|excel|xlsx' //只允许上传压缩文件
            ,done: function(res){
                if(res.code==1){
                    layer.msg('上传成功,已解析数据',{icon:6});
                }else{
                    layer.msg('解析失败',{icon:5});
                }
            }
        });
        form.on('submit(formsub)',function(data){
            layer.msg('导入数据具体详情未协商确认,待确认后处理');
            return false;
        })
    })
</script>
```

在上传后的sale/do_upload中去进行解析上传的excel

```php
public function do_upload(){
    //引入文件	
    \think\Loader::import('PHPExcel.PHPExcel');
    //\think\Loader::import('PHPExcel.Classes.PHPExcel');

    $objPHPExcel = new \PHPExcel();
    //获取表单上传文件
    $file = request()->file('file');
    $info = $file->validate(['ext' => 'xlsx,xls'])->move(ROOT_PATH . 'public' . DS . 'uploads');
    //数据为空返回错误
    if(empty($info)){
        $output['status'] = false;
        $output['info'] = '导入数据失败~';
        $this->ajaxReturn($output);
    }
    //获取文件名
    $exclePath = $info->getSaveName();
    //上传文件的地址
    $filename = ROOT_PATH . 'public' . DS . 'uploads'.DS . $exclePath;
    $extension = strtolower( pathinfo($filename, PATHINFO_EXTENSION) );
    \think\Loader::import('PHPExcel.IOFactory.PHPExcel_IOFactory');
    //\think\Loader::import('PHPExcel.Classes.PHPExcel.IOFactory.PHPExcel_IOFactory');

    if ($extension =='xlsx') {
        $objReader = new \PHPExcel_Reader_Excel2007();
        $objExcel = $objReader ->load($filename);
    } else if ($extension =='xls') {
        $objReader = new \PHPExcel_Reader_Excel5();
        $objExcel = $objReader->load($filename);
    }

    $excel_array=$objExcel->getsheet(0)->toArray();   //转换为数组格式
    array_shift($excel_array);  //删除第一个数组(标题);
    array_shift($excel_array);  //删除th

    $data=[];
    foreach ($excel_array as $k=>$v){
        $data[$k]["danhao"]=$v[0];//单号
        $data[$k]["type_name"]=$v[1];//类型名称
        $data[$k]["name"]=$v[2];
        $data[$k]["number"]=$v[3];
        $data[$k]["price"]=$v[4];
        $data[$k]["danwei"]=$v[0];
        $data[$k]["create_user"]=$v[5];
        $data[$k]["create_time"]=$v[6];
        $data[$k]["remark"]=$v[7];
    }

    $msg=[
        'code'=>1,
        'msg'=>'已获取信息',
    ];
    $msg['data']['src']=$filename;
    $msg['data']['data']=$data;

    return json_encode($msg);
}
```
