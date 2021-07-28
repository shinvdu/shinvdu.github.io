---
layout: post
title:  "解决drupal8下面email 样式发送的问题, 以发送原始邮件(how to send raw email in dupal8 without any style and tag filter)"
date:   2021-07-27 10:28:09 +0800
categories: Note
---
我终于搞定了。 太不容易了， 用xdebug查了好久，才debug出来。 

#### 问题(problem):  

最近发现时间轴的总是有样式问题，  drupal默认对模板作了两次处理。  同样一个模板的， 我用nodejs来发就正常， 用drupal来发， 样式就出问题。 drupal默认对原始邮件的模板的样式和标签进行了过滤


#### 问题分析(root cause):

##### 1. drupal内核代码进行了过滤（drupal core code have do some filter). 

以下是处理逻辑代码的位置， Html::transformRootRelativeUrlsToAbsolute会把数据转成dom对象， 再输出， 这中间会把样式和特殊的标签给去掉。 

/pathtodrupal/core/lib/Drupal/Core/Mail/MailManager.php

line number 278: 
```
    // Retrieve the responsible implementation for this message.
    $system = $this->getInstance(['module' => $module, 'key' => $key]);

    // Attempt to convert relative URLs to absolute.
    foreach ($message['body'] as &$body_part) {
      if ($body_part instanceof MarkupInterface) {
        $body_part = Markup::create(Html::transformRootRelativeUrlsToAbsolute((string) $body_part, \Drupal::request()->getSchemeAndHttpHost()));
      }
    }

    // Format the message body.
    $message = $system->format($message);

```

/pathtodrupal/core/lib/Drupal/Component/Utility/Html.php

```
  public static function transformRootRelativeUrlsToAbsolute($html, $scheme_and_host) {
    assert(empty(array_diff(array_keys(parse_url($scheme_and_host)), ["scheme", "host", "port"])), '$scheme_and_host contains scheme, host and port at most.');
    assert(isset(parse_url($scheme_and_host)["scheme"]), '$scheme_and_host is absolute and hence has a scheme.');
    assert(isset(parse_url($scheme_and_host)["host"]), '$base_url is absolute and hence has a host.');

    $html_dom = Html::load($html);
    $xpath = new \DOMXpath($html_dom);
...
    return Html::serialize($html_dom);
  }

}

```

##### 2. drupal formater have also filter the tag. 

在我的应用中， 我用了swiftmail formater, 它也去掉了特殊标签和样式。 

/pathtodrupal/modules/contrib/swiftmailer/src/Plugin/Mail/SwiftMailer.php


#### 如何解决(how to solve)

##### 1. 保证传给发邮件函数的body为string, 而不是markup对象(passing string instead of markup object)

```
function mymodule_sendmail($to, $subject, $body, $params = []) {

  $from = \Drupal::config('system.site')->get('mail');
  $params['subject'] = $subject;
  $params['body'] = $body;
  $langcode = \Drupal::languageManager()->getDefaultLanguage();

  // Send email with drupal_mail.
  return \Drupal::service('plugin.manager.mail')->mail('wristcheck_mail', 'wristcheck_mail', $to, $langcode, $params, $from);
}

```

保证body为string, 而不是markup对象

#####  2. 自己写一个不对邮件过滤的formater(write a self formater)

最后， 是自己写了一个emal formater， 替代了内置的formater了， 不做处理， 这样才解决。 

/pathtodrupal/modules/custom/mymodule/src/Plugin/Mail/MymoduleMailer.php

```
<?php

namespace Drupal\mymodule\Plugin\Mail;

use Drupal\Core\Mail\MailInterface;


/**
 * Provides a 'Mymodule Mailer' plugin to send emails.
 *
 * @Mail(
 *   id = "mymodulemailer",
 *   label = @Translation("Mymodule Mailer"),
 *   description = @Translation("Mymodule Mailer Plugin.")
 * )
 */
class MymoduleMailer implements MailInterface {


  /**
   * Formats a message composed by drupal_mail().
   *
   * @param array $message
   *   A message array holding all relevant details for the message.
   *
   * @return array
   *   The message as it should be sent.
   */
  public function format(array $message) {
    $text = '';
    foreach ($message['body'] as $body) {
      $text = $text . $body;
    }
    $message['body'] = $text;
    return $message;
  }

  /**
   * Sends a message composed by drupal_mail().
   *
   * @param array $message
   *   A message array holding all relevant details for the message.
   *
   * @return bool
   *   TRUE if the message was successfully sent, and otherwise FALSE.
   */
  public function mail(array $message) {
    return FALSE;
  }


}
```


