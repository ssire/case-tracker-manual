# Media engine

The media engine is both a template library to generate e-mail messages from e-mail templates and variables definitions, and a media library with helper XQuery functions to send and trace e-mail messages. 

## The e-mail template engine

The e-mail template engine is available in `lib/mail.xqm`.

E-mail templates must be deployed in the `global-information/email.xml` resource in the application data collection.

Variable for template injection are defined in `variables.xml` in the application configuration.

Example of an e-mail message template :

```xml
<Alert Name="activity-spontaneous-alert" Lang="en">
  <Subject>Message about @@Activity_Title@@ coaching activity</Subject>
  <Message>
    <Text>Enter your own message here...</Text>
    <Text>Direct link to Company coaching activity on Case Tracker: @@Link_To_Activity@@</Text>
    <Text>@@User_First_Name@@ @@User_Last_Name@@</Text>
  </Message>
</Alert>
```

Example of a variable definition :

```xml
<Variable>
  <Name>Project_Acronym</Name>
  <Expression>$case/Information/Acronym/text()</Expression>
</Variable>
```

TO BE COMPLETED

## The media library

The media library is available in `lib/media.xqm`. 

The *media:send-email* function is a wrapper around the eXist-DB *mail:send-email* function to send e-mail messages. It adds a layer for tracing e-mail for debug purpose and to configure the e-mail server. They are controller with an XML media configuration vocabulary used in the application configuration.

The tracing function dumps a copy of every message and message headers to the `/db/debug/debug.xml` resource. To activate it you need to :

* store a `debug.xml` resource with a root `Debug` element into the `/db/debug` collection, the file must be writable at least by the *users* group (or by guest user if you are liberal)
* configure the message category to dump into the application settings (see below)

The media library associates each message with a *category*. This is a string token that you should give as a parameter when invoking *send-email*. Its purpose is to allow a category by category control on the tracing functionality. You can for instance use the category to identify the functional module at the origin of the message (e.g. *workflow*, *account*, *action*, *reminder*).

Note that to send e-mail the Mail module must be activated in eXist-DB `conf.xml` file.

### The media configuration vocabulary

The media configuration must be placed into the application settings file (aka `settings.xml` file).

The `SMTPServer` element defines the server name of the SMTP server to contact for sending e-mail messages. The library uses an anonymous connexion on port 25 (no authentification), so your SMTP server must be configured accordingly. Note that you can start the server name with ! (bang) to deactivate the effective sending of the e-mail. In that case the *send-email* function will execute normally but no e-mail message will be sent. This is useful for instance on a development machine where you do not have an SMTP server.

The `DefaultEmailSender` element defines the value that will be copied to the *from* e-mail SMTP header. Note that it should be an e-mail address on the same domain name as the domain from which your SMTP server will operate otherwise you e-mail may be rejected as spam. The *from* parameter of the *send-email* function will instead be set as the *reply-to* field of the e-mail message. This way users will be able to use the reply button of their e-mail graphical client altghough they will see the message originating from the *DefaultEmailSender*.

You must note that to be sure that your application e-mail messages will arrive at destination and not be treated as spam, you should configure an SPF DNS record and a PTR DNS record on your domain name, check that with your ISP.

You can check the SPF record using the command : `nslookup -type=txt your-domain.extension`

You can check the PTR record using the command : `dig -x reverse.ip.address.of.your.domain`

The `Media` element allows to configures individually each category :

* the `Allow` section must explicitely define one `Category` element for each category of message which are really sent
  * this implements a second fine-grain layer of control over the ! (bang) notation in the the `SMTPServer` element
* the `Debug` section defines one `Category` element for each category of message to trace

So basically if you configure a `Category` only in the `Debug` section, all the messages from this category will be traced in the `debug.xml` file if present in `/db/debug`, but they won't be actually sent. 

The case tracker uses at least the following categories :

- workflow : notification sent by workflow status change
- account : messages related to account management (login creation, etc.) 
- reminders : notifications sent by the alert module
- action : notifications sent by other events (e.g. when saving a document that contains a certain value, like an approval date for an internal control process)

Example configuration :

```xml
<Settings>
  <SMTPServer>!localhost</SMTPServer>
  <DefaultEmailSender>case-tracker-no-reply@coachcom2020.eu</DefaultEmailSender>
  <Media>
    <Allow>
      <Category>account</Category>
    </Allow>
    <Debug>
      <Category>reminders</Category>
      <Category>account</Category>
      <Category>workflow</Category>
      <Category>action</Category>
    </Debug>
  </Media>
</Settings>
```

The configuration above will not really send any e-mail message because of the ! (bang) in `SMTPServer`, but it will trace 4 categories of e-mail messages (reminders, account, workflow, action) into the *debug.xml* file

### The e-mail snippet

In case you have any doubt with the correct functioning of the SMTP server on your production server, here is a sample XQuery file to run on the eXist-DB instance running on that server (you can for instance copy/paste it into the Java admin remote code execution window, execute it from the sandbox or use bin/client.sh -F *filename.xql*). Do not forget to edit the `to`, `cc` and `reply-to` fields with an e-mail address you can access to check the e-mail : 

```xquery
xquery version "1.0";
import module namespace xdb = "http://exist-db.org/xquery/xmldb";
import module namespace request="http://exist-db.org/xquery/request";
import module namespace util="http://exist-db.org/xquery/util";
import module namespace mail = "http://exist-db.org/xquery/mail";

declare function local:report-exception( $to as xs:string ) as xs:string
{
 let $err :=  concat("Failed to send email message : ", $util:exception-message)
 return
   $err
};

<Test>
  {
    <Send>
      {
      let $to-send :=
        <mail>
          <from>case-tracker-no-reply@coachcom2020.eu</from>
          <to>john@doe.com</to>
          <cc>foo@bar.com</cc>
          <reply-to>doe@john.com</reply-to>
          <subject>E-mail notification test</subject>
          <message>
              <text>Dear,

If you can read me then it worked !
              </text>
          </message>
        </mail>
        let $server := "localhost"
        return 
            let $res := util:catch('*', mail:send-email($to-send , $server, ()), local:report-exception("dummy"))
            return 
              if ($res eq true()) then 
                <Done>{ $res }</Done>
              else if ($res castable as xs:string) then
                <Exception>{ $res }</Exception>
              else
                <Unkown>{$res}</Unkown>
      }
    </Send>
  }
</Test>
```
