# เขียนโค้ดรับไฟล์ audio จาก LINE Server

ข้อความที่ LINE Server ส่งมาให้เรา

```
{
   "events":[
      {
         "replyToken":"nHuyWiB7yP5Zw52FIkcQobQuGDXCTA",
         "type":"message",
         "timestamp":1462629479859,
         "source":{
            "type":"user",
            "userId":"U206d25c2ea6bd87c17655609a1c37cb8"
         },
         "message":{
            "id":"325708",
            "type":"audio"
         }
      }
   ]
}
```

message ที่ LINE ส่งให้เรานั้นจะเห็นว่า แค่สั่ง id กับ type มาให้ ไม่ส่งชื่อไฟล์ นามสกุลไฟล์ นั่นเพราะว่า LINE อนุญาติให้ส่งไฟล์เสียงได้เฉพาะนามสกุล m4a เท่านั้น ในกรณีเราต้องการเก็บไฟล์เสียงไว้ใช้งานในภายหลังเราจะต้องทำการ get content ของไฟล์ออกมา ซึ่ง LINE มีคำสั่งให้ใช้งานอยู่ \(ตัวอย่างอยู่ในโค้ด\)

\*\*\* สิ่งที่ต้องจำ LINE Server จะลบไฟล์ทิ้งเมื่อถึงช่วงระยะเวลาหนึ่ง เพราะฉะนั้นถ้าต้องการเก็บไฟล์ไว้ จะต้องเขียนไฟล์เก็บไว้ที่เซิฟเวอร์ตัวเอง

## เปิดไฟล์ index.php ขึ้นมาแล้วเขียนโค้ดดังนี้

```php
require_once('./vendor/autoload.php');

$channel_token = '1v2OUa9tuMIiDhEg57ANbsRaBDbBGP9nlCC+Dpvt5HrsQ+LqcrImWPUBkH8re/pwqxv56d15kZeMoU/vQ0zuzPFlbhFM7AhRMZwLrSkLdcjbFurwXGOyHLt8MdgzLfAe7r0BsQV5cATlUanW3OgJewdB04t89/1O/w1cDnyilFU=';
$channel_secret = '9b2c7349ea939ef723a3cb453d774c86';

// Get message from Line API
$content = file_get_contents('php://input');
$events = json_decode($content, true);

if (!is_null($events['events'])) {

	// Loop through each event
	foreach ($events['events'] as $event) {
    
        // Line API send a lot of event type, we interested in message only.
		if ($event['type'] == 'message') {

            // Get replyToken
            $replyToken = $event['replyToken'];
            
            switch($event['message']['type']) {
                
                case 'audio':
                
                    /*    
                    $messageID = $event['message']['id'];
                                  
                    $httpClient = new \LINE\LINEBot\HTTPClient\CurlHTTPClient($channel_token);
                    $bot = new \LINE\LINEBot($httpClient, array('channelSecret' => $channel_secret));
                    $response = $bot->getMessageContent($messageID);

                    if ($response->isSucceeded()) {
                        $tempfile = tmpfile();
                        fwrite($tempfile, $response->getRawBody());
                    }

                    // Returns status code 200 and the content in binary.
                    // Note: Content is automatically deleted after a certain period from when the message was sent.
                    // There is no guarantee for how long content is stored.
                    */

                    // Reply message
                    $respMessage = 'We received your audio';
            
                    break;
                default:
                    // Reply message
                    $respMessage = 'Please send audio only';
                    break;
            }

            $httpClient = new \LINE\LINEBot\HTTPClient\CurlHTTPClient($channel_token);
            $bot = new \LINE\LINEBot($httpClient, array('channelSecret' => $channel_secret));

            $textMessageBuilder = new \LINE\LINEBot\MessageBuilder\TextMessageBuilder($respMessage);
            $response = $bot->replyMessage($replyToken, $textMessageBuilder);
		}
	}
}

echo "OK";

```

เสร็จแล้ว push โค้ดขึ้น repository รอสักแป้บให้ทาง Heroku ดึงโค้ดส่งขึ้นโฮสต์ก่อน ลองส่งไฟล์เสียงสั้นๆอะไรก็ได้ให้บอทดู

## อธิบายโค้ด

```php
require_once('./vendor/autoload.php');
```

ทำการ include ไฟล์ LINEBot SDK เข้ามา

```php
$channel_token = '1v2OUa9tuMIiDhEg57ANbsRaBDbBGP9nlCC+Dpvt5HrsQ+LqcrImWPUBkH8re/pwqxv56d15kZeMoU/vQ0zuzPFlbhFM7AhRMZwLrSkLdcjbFurwXGOyHLt8MdgzLfAe7r0BsQV5cATlUanW3OgJewdB04t89/1O/w1cDnyilFU=';
$channel_secret = '9b2c7349ea939ef723a3cb453d774c86';
```

ป้อน Channel token กับ Channel secret ที่ได้มาจาก LINE API เข้าไป อันนี้ของใครของมันไม่เหมือนกัน ถ้าหากคุยกับบอทแล้วบอทไม่ตอบอะไร ให้ดูตรงนี้ให้มั่นใจว่าถูกต้อง

```php
$content = file_get_contents('php://input');
$events = json_decode($content, true);
```

get ค่าที่ LINE Server ส่งมาให้แล้วแปลงจาก JSON ไปเป็น Array

```php
case 'audio':
```

ตรวจสอบ type ว่าเป็น audio

```php
$respMessage = 'We received your audio';
```

ตอบกลับไปว่าเราได้รับไฟล์เสียงของคุณแล้ว

```
/*   
$messageID = $event['message']['id'];
              
$httpClient = new \LINE\LINEBot\HTTPClient\CurlHTTPClient($channel_token);
$bot = new \LINE\LINEBot($httpClient, array('channelSecret' => $channel_secret));
$response = $bot->getMessageContent($messageID);

if ($response->isSucceeded()) {
    $tempfile = tmpfile();
    fwrite($tempfile, $response->getRawBody());
}

// Returns status code 200 and the content in binary.
// Note: Content is automatically deleted after a certain period from when the message was sent.
// There is no guarantee for how long content is stored.
*/
```

ถ้าหากต้องการเก็บไฟล์ไว้ในเครื่องเซิฟเวอร์ ให้เปิดคอมเม้นท์โค้ดชุดนี้ แต่ก็ต้องมั่นใจนะว่าโฟลเดอร์มี permission สามารถเขียนไฟล์ได้

```php
$httpClient = new \LINE\LINEBot\HTTPClient\CurlHTTPClient($channel_token);
$bot = new \LINE\LINEBot($httpClient, array('channelSecret' => $channel_secret));

$textMessageBuilder = new \LINE\LINEBot\MessageBuilder\TextMessageBuilder($respMessage);
$response = $bot->replyMessage($replyToken, $textMessageBuilder);
```

LINE SDK สำหรับส่งข้อความกลับไปหายูสเซอร์

P

