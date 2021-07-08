<h1 align="center">OrziceLib</h1>

<p align="center">
Orzice经常使用的PHP库和功能组件，部分是自己造的轮子
</p>

## 安装
```
composer require orzice/orzicelib
```

## 代码仓库

* GitHub地址：[https://github.com/orzice/orzicelib](https://github.com/orzice/orzicelib)

* Gitee地址：[https://gitee.com/orzice/orzicelib](https://gitee.com/orzice/orzicelib)

## ThinkPHP - 好用的库

我经常使用 ThinkPHP 相关的代码，这些是TP相关的库

* 配置
```
use Orzice\Lib\thinkphp\;
```


### Cron - 计划任务

* 配置
```
use Orzice\Lib\thinkphp\cron\Cron;
```

```
config/console.php

return [
    // 指令定义
    'commands' => [
		'cron' => 'app\common\command\Cron',
    ],
];

```

* 调用方式[例子代码]

```
// app\common\command\Cron.php 文件
// php think cron

namespace app\common\command;

use think\console\Command;
use think\console\Input;
use think\console\input\Argument;
use think\console\input\Option;
use think\console\Output;

use think\facade\Event;
use Orzice\Lib\thinkphp\cron\Cron;

class Cron extends Command
{
    /**
     * @static
     * @var array Saves all the cron jobs
     */
    private static $cronJobs = array();


    protected function configure()
    {
        $this->setName('cron')
            ->setDescription('系统计划任务服务（By：Orzice）');
    }

    protected function execute(Input $input, Output $output)
    {
        $output->writeln("");
        $output->writeln("---".date('Y-m-d H:i:s')."---");
        //-------------
        //在这里创建任务
        // ....
        //-------------
        $output->writeln("定时任务服务启动" );
        $output->writeln("");

        Event::trigger('cron.collectJobs');

        $report = Cron::run();

        if($report['inTime'] === -1) {
            $inTime = -1;
        } else if ($report['inTime']) {
            $inTime = 'true';
        } else {
            $inTime = 'false';
        }

        $output->writeln("");
        $output->writeln("---".date('Y-m-d H:i:s')."---");
        $output->writeln("Run date :" .$report['rundate']);
        $output->writeln("In time :" .$inTime);
        $output->writeln("Run time :" .round($report['runtime'], 4));
        $output->writeln("Errors :" .$report['errors']);
        $output->writeln("Jobs :" .count($report['crons']));
        $output->writeln("-------------------------");

    }

}
```

* 创建任务

```
use think\facade\Event;
use Orzice\Lib\thinkphp\cron\Cron;

Event::listen('cron.collectJobs', function () {
    Cron::add('Name', '*/1 * * * * *', function () {
    //您的任务脚本 每分钟1次
    return;
    });
});
```

### Upload - 上传文件（本地 / 阿里云 /  七牛云 / 腾讯云）

* 配置
```
use Orzice\Lib\thinkphp\upload\Uploadfile;
```
* 配置 需要数据库表

```
DROP TABLE IF EXISTS `member_uploadfile`;
CREATE TABLE `member_uploadfile` (
  `id` int(20) unsigned NOT NULL AUTO_INCREMENT COMMENT 'ID',
  `upload_type` varchar(20) CHARACTER SET utf8 NOT NULL DEFAULT 'local' COMMENT '存储位置',
  `original_name` varchar(255) CHARACTER SET utf8 DEFAULT NULL COMMENT '文件原名',
  `url` varchar(255) CHARACTER SET utf8 NOT NULL DEFAULT '' COMMENT '物理路径',
  `file` varchar(255) CHARACTER SET utf8 NOT NULL COMMENT '真实物理路径',
  `image_width` varchar(30) CHARACTER SET utf8 NOT NULL DEFAULT '' COMMENT '宽度',
  `image_height` varchar(30) CHARACTER SET utf8 NOT NULL DEFAULT '' COMMENT '高度',
  `image_type` varchar(30) CHARACTER SET utf8 NOT NULL DEFAULT '' COMMENT '图片类型',
  `image_frames` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '图片帧数',
  `mime_type` varchar(100) CHARACTER SET utf8 NOT NULL DEFAULT '' COMMENT 'mime类型',
  `file_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '文件大小',
  `file_ext` varchar(100) CHARACTER SET utf8 DEFAULT NULL,
  `sha1` varchar(40) CHARACTER SET utf8 NOT NULL DEFAULT '' COMMENT '文件 sha1编码',
  `create_time` int(10) DEFAULT NULL COMMENT '创建日期',
  `update_time` int(10) DEFAULT NULL COMMENT '更新时间',
  `upload_time` int(10) DEFAULT NULL COMMENT '上传时间',
  `uid` int(10) NOT NULL DEFAULT '0' COMMENT '上传者',
  `state` int(10) NOT NULL DEFAULT '0' COMMENT '0为临时 1为正式',
  PRIMARY KEY (`id`),
  KEY `upload_type` (`upload_type`),
  KEY `original_name` (`original_name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 ROW_FORMAT=COMPACT COMMENT='用户上传文件表';
```

* 调用方式[例子代码]

```
use Orzice\Lib\thinkphp\upload\Uploadfile;

$upload = Uploadfile::instance()
                ->setUploadType($data['upload_type'])
                ->setUploadConfig($uploadConfig)
                ->setFile($data['file'])
                ->setUid($adminId)
                ->setState(0)
                ->save();
```

* 相关参数解析

```
$uploadConfig = array (
  'upload_type' => 'local',
  'upload_allow_ext' => 'doc,gif,ico,icon,jpg,mp3,mp4,p12,pem,png,rar,jpeg,zip',
  'upload_allow_size' => '1048576',
  'upload_allow_mime' => 'image/gif,image/jpeg,video/x-msvideo,text/plain,image/png',
  'upload_allow_type' => 'local,alioss,qnoss,txcos',
  'alioss_access_key_id' => '填你的',
  'alioss_access_key_secret' => '填你的',
  'alioss_endpoint' => '填你的',
  'alioss_bucket' => '填你的',
  'alioss_domain' => '填你的',
  'txcos_secret_id' => '填你的',
  'txcos_secret_key' => '填你的',
  'txcos_region' => '填你的',
  'tecos_bucket' => '填你的',
  'qnoss_access_key' => '填你的',
  'qnoss_secret_key' => '填你的',
  'qnoss_bucket' => '填你的',
  'qnoss_domain' => '填你的',
)

```

## Wechat - 微信相关

未设置


## Request - 请求相关

未设置
