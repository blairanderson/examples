---
layout: post
title:  Slack Incoming Webhook
service: slack
language: javascript
resource: true
date:   2015-04-09 10:00:00
---

First visit [https://my.slack.com/services/new/incoming-webhook](https://my.slack.com/services/new/incoming-webhook)
and create a webhook.

{% highlight javascript %}

var resultContainer = $('#slack-webhook-test'), // this will help you know if the webhook works...

  url = $("#slack-webhook-url").val(), // this is the url

  data = {
    payload: JSON.stringify({text: "This is a test from" + location.href})
  };

if (url && url.indexOf("https://hooks.slack.com/services") > -1) {

  $.post(url, data).always(function(response,status) {

    if (status === "error") {
      response = response.responseText;
    } else {
      response = response + ". Please click save!"
    }

    resultContainer.text("["+status+"]: "+response);
  });

} else {

  resultContainer.text("Enter a Slack Webhook URL!").fadeIn('fast');

}

{% endhighlight %}
