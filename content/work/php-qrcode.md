---
title: "PHP生成二维码"
date: 2018-07-01T23:32:44+08:00
categories: ["PHP"]
tags: ["PHP", "二维码"]
---

工作中遇到生成订单使用二维码支付的场景，从网上搜集资料mark一下

#### 1.利用Google API生成二维码
Google提供了较为完善的二维码生成接口，调用API接口简单，然而国内你懂的，示例代码：
```
$urlToEncode='https://www.icodego.com'; 
generateQRfromGoogle($urlToEncode); 
/** 
 * google api 二维码生成【QRcode可以存储最多4296个字母数字类型的任意文本，具体可以查看二维码数据格式】 
 * @param string $chl 二维码包含的信息，可以是数字、字符、二进制信息、汉字。 
 不能混合数据类型，数据必须经过UTF-8 URL-encoded 
 * @param int $widhtHeight 生成二维码的尺寸设置 
 * @param string $EC_level 可选纠错级别，QR码支持四个等级纠错，用来恢复丢失的、读错的、模糊的、数据。 
 * L-默认：可以识别已损失的7%的数据 
 * M-可以识别已损失15%的数据 
 * Q-可以识别已损失25%的数据 
 * H-可以识别已损失30%的数据 
 * @param int $margin 生成的二维码离图片边框的距离 
 */
function generateQRfromGoogle($chl,$widhtHeight ='150',$EC_level='L',$margin='0') 
{ 
 $chl = urlencode($chl); 
 echo '<img src="http://chart.apis.google.com/chart?chs='.$widhtHeight.'x'.$widhtHeight.' 
 &cht=qr&chld='.$EC_level.'|'.$margin.'&chl='.$chl.'" alt="QR code" widhtHeight="'.$widhtHeight.'
 " widhtHeight="'.$widhtHeight.'"/>'; 
} 
```

#### 2.使用PHP类库phpqrcode生成二维码
phpqrcode是一个PHP二维码生成类库，利用它可以轻松生成二维码，官网提供了下载和多个演示demo，查看地址：[http://phpqrcode.sourceforge.net](http://phpqrcode.sourceforge.net/ "qrcode")  
下载之后只需要使用phpqrcode.php就可以生成二维码，需要注意的是PHP环境必须开启支持GD2  
phpqrcode.php提供了一个关键的png()方法，其中参数$text表示生成二维码的信息文本；参数$outfile表示是否输出二维码图片文件，默认否；参数$level表示容错率，也就是有被覆盖的区域还能识别，分别是 L（QR_ECLEVEL_L，7%），M（QR_ECLEVEL_M，15%），Q（QR_ECLEVEL_Q，25%），H（QR_ECLEVEL_H，30%）； 参数$size表示生成图片大小，默认是3；参数$margin表示二维码周围边框空白区域间距值；参数$saveandprint表示是否保存二维码并显示
```
public static function png($text, $outfile=false, $level=QR_ECLEVEL_L, $size=3, $margin=4, $saveandprint=false) 
{
    $enc = QRencode::factory($level, $size, $margin); 
    return $enc->encodePNG($text, $outfile, $saveandprint=false); 
}
```
使用方式
```
include 'phpqrcode.php';   
QRcode::png('https://www.icodego.com');
die;
```
二维码中间加logo应用  
原理：先使用phpqrcode生成一张二维码图片，再利用php的image相关函数，将事先准备好的logo图片加入到刚生成的原始二维码图片中间，然后重新生成一张新的二维码图片。
```
include 'phpqrcode.php'; 
$value = 'https://www.icodego.com'; //二维码内容 
$errorCorrectionLevel = 'L';//容错级别 
$matrixPointSize = 6;//生成图片大小 
//生成二维码图片 
QRcode::png($value, 'qrcode.png', $errorCorrectionLevel, $matrixPointSize, 2); 
$logo = 'logo.png';//准备好的logo图片 
$QR = 'qrcode.png';//已经生成的原始二维码图 
  
if ($logo !== FALSE) {
    $QR = imagecreatefromstring(file_get_contents($QR)); 
    $logo = imagecreatefromstring(file_get_contents($logo)); 
    $QR_width = imagesx($QR);//二维码图片宽度 
    $QR_height = imagesy($QR);//二维码图片高度 
    $logo_width = imagesx($logo);//logo图片宽度 
    $logo_height = imagesy($logo);//logo图片高度 
    $logo_qr_width = $QR_width / 5; 
    $scale = $logo_width/$logo_qr_width; 
    $logo_qr_height = $logo_height/$scale; 
    $from_width = ($QR_width - $logo_qr_width) / 2; 
    //重新组合图片并调整大小 
    imagecopyresampled($QR, $logo, $from_width, $from_width, 0, 0, $logo_qr_width, 
    $logo_qr_height, $logo_width, $logo_height); 
} 
//输出图片 
imagepng($QR, 'result.png'); 
echo '<img src="result.png">'; 
```

#### 3.libqrencode

#### 4.QRcode Perl CGI & PHP scripts
地址：http://www.swetake.com/qr/qr_cgi.html