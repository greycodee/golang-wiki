# net/smtp包

## 简介
`SMTP`包是实现`SMTP(Simple Mail Transfer Protocol)`协议的一个包,遵守了[RFC5321](https://www.rfc-editor.org/rfc/rfc5321.html),同时相关拓展也准守了相关`RFC文档`

> 8BITMIME    [RFC 1652](https://rfc-editor.org/rfc/rfc1652.html) </br>
> AUTH        [RFC 2554](https://rfc-editor.org/rfc/rfc2554.html)</br>
> STARTTLS    [RFC 3207](https://rfc-editor.org/rfc/rfc3207.html)

## 函数

### SendMail
`smtp`包封装了一个发送邮件的方法[SendMail](https://go.googlesource.com/go/+/go1.16/src/net/smtp/smtp.go#323)，调用这个方法可以直接发送邮件，这个方法里面按照`smtp`协议的请求步骤进行了封装，所以调用者不必了解`smtp`协议的具体发送步骤就可以直接发送邮件。`SendMail`函数和`net/smtp`软件包不支持`DKIM签名`，`MIME附件`（请参阅[mime/multipart](https://pkg.go.dev/mime/multipart)软件包）或其他邮件功能。更高级别的程序包存在于标准库之外

```go
func SendMail(addr string, a Auth, from string, to []string, msg []byte) error
```

<details>
<summary style="color:#42b983">查看示例</summary>

```go
package main

import (
	"log"
	"net/smtp"
)

func main() {
	// 设置PlainAuth验证的账号和smtp服务器信息
	auth := smtp.PlainAuth("", "user@example.com", "password", "mail.example.com")

	// 设置发送人，数组里的每个邮件地址都会进行RCPT调用
	to := []string{"recipient@example.net"}

    // 编写发送的消息
	msg := []byte("To: recipient@example.net\r\n" +
		"Subject: discount Gophers!\r\n" +
		"\r\n" +
		"This is the email body.\r\n")

    // 调用函数发送邮件
	err := smtp.SendMail("mail.example.com:25", auth, "sender@example.org", to, msg)
	if err != nil {
		log.Fatal(err)
	}
}
```
</details>

!> 上面的`msg`里的内容需准守`smtp`协议的内容规范，`To`代码接收邮件的人，`Subject`代表邮件的主题。内容部分以符号`.`结尾。</br>参考[RFC 822](https://www.rfc-editor.org/rfc/rfc822.html)
> Tips:发送“密件抄送”消息的方法是，在to参数中包括电子邮件地址，但在msg标头中不包括该电子邮件地址。也就是说只进行`RCPT`调用，而不在消息中注明该地址。

## Auth接口
[Auth](https://go.googlesource.com/go/+/go1.16/src/net/smtp/auth.go#15)接口有两个方法，`Start`和`Next`。
- `Start`方法表示开始开始对服务器进行身份验证。它返回**认证协议的名称**，以及可选地包含在发送到服务器的初始`AUTH`消息中的数据。它可以返回`proto ==“”`来表示跳过身份验证。如果返回的`error`不为`nil`，则`SMTP`客户端会终止身份验证尝试并关闭连接。

- `Next`方法表示接下来继续进行身份验证，服务器发送`formServer`数据，当`more`字段为`true`时，表示希望接收响应数据，此时数据以`[]byte`数据格式返回。当`more`字段为`false`时，返回`nil`。如果返回的`error`不为`nil`，则`SMTP`客户端会终止身份验证尝试并关闭连接。

<details>
<summary style="color:#42b983">查看Auth接口源码</summary>

``` go
type Auth interface {

    Start(server *ServerInfo) (proto string, toServer []byte, err error)

    Next(fromServer []byte, more bool) (toServer []byte, err error)
}
```
</details>

### CRAMMD5Auth
`CRAMMD5Auth`返回实现[RFC 2195](https://rfc-editor.org/rfc/rfc2195.html)中定义的`CRAM-MD5`身份验证机制的`Auth` 。

`CRAMMD5Auth`实现了`Auth`接口的两个方法
<details>
<summary style="color:#42b983">点击查看CRAMMD5Auth源码</summary>

```go
type cramMD5Auth struct {
	username, secret string
}
// 提供外部调用的方法，传入username和secret,返回的Auth使用给定的用户名和密码使用质询-响应机制对服务器进行身份验证。
func CRAMMD5Auth(username, secret string) Auth {
	return &cramMD5Auth{username, secret}
}
// 实现Auth的Start方法，返回协议名`CRAM-MD5`
func (a *cramMD5Auth) Start(server *ServerInfo) (string, []byte, error) {
	return "CRAM-MD5", nil, nil
}
// 实现Auth的Next方法，进行加密
func (a *cramMD5Auth) Next(fromServer []byte, more bool) ([]byte, error) {
	if more {
		d := hmac.New(md5.New, []byte(a.secret))
		d.Write(fromServer)
		s := make([]byte, 0, d.Size())
		return []byte(fmt.Sprintf("%s %x", a.username, d.Sum(s))), nil
	}
	return nil, nil
}
```
</details>

### PlainAuth

`PlainAuth`返回实现[RFC 4616](https://rfc-editor.org/rfc/rfc4616.html)定义的`Auth`。

!> 仅当连接使用`TLS`或`连接到本地主机`时，`PlainAuth`才会发送凭据。否则，身份验证将失败并显示错误，而不发送凭据。

`PlainAuth`实现了`Auth`接口的两个方法

<details>
<summary style="color:#42b983">查看PlainAuth源码</summary>

```go
type plainAuth struct {
	identity, username, password string
	host                         string
}
// 暴露给外部调用的方法，一般identity为空字符串。传入账号名，密码，和smtp服务器地址
func PlainAuth(identity, username, password, host string) Auth {
	return &plainAuth{identity, username, password, host}
}
// 判断是否是本地地址
func isLocalhost(name string) bool {
	return name == "localhost" || name == "127.0.0.1" || name == "::1"
}
// 实现Auth接口的Start方法，返回PLAIN协议和发送到服务器的初始AUTH消息中的数据
func (a *plainAuth) Start(server *ServerInfo) (string, []byte, error) {
	if !server.TLS && !isLocalhost(server.Name) {
		return "", nil, errors.New("unencrypted connection")
	}
	if server.Name != a.host {
		return "", nil, errors.New("wrong host name")
	}
	resp := []byte(a.identity + "\x00" + a.username + "\x00" + a.password)
	return "PLAIN", resp, nil
}

func (a *plainAuth) Next(fromServer []byte, more bool) ([]byte, error) {
	if more {
		return nil, errors.New("unexpected server challenge")
	}
	return nil, nil
}
```

</details>

## Client结构体

<details>
<summary style="color:#42b983">查看Client结构体</summary>

```go
type Client struct {
	// 客户端使用的textproto.Conn。它被导出以允许客户端添加扩展。
	Text *textproto.Conn
	// 保留一个对连接的引用，以便于以后创建TLS链接
	conn net.Conn
	// 客户端是否正在使用TLS
	tls        bool
	serverName string
	// 支持的拓展的Map
	ext map[string]string
	// 支持auth的机制
	auth       []string
	localName  string
    // 是否已经调用 HELLO/EHLO
	didHello   bool   
    // HELO响应的错误
	helloError error
}
```

</details>

### Dial
此函数将会调用[[net.Dial](https://go.googlesource.com/go/+/go1.16/src/net/dial.go#317)函数，与传入的`SMTP`地址（*地址需包含端口，例如：smtp.qq.com:25*）建立`TCP`链接，然后通过调用[NewClient](https://go.googlesource.com/go/+/go1.16/src/net/smtp/smtp.go#62)函数返回一个[SMTP客户端](https://go.googlesource.com/go/+/go1.16/src/net/smtp/smtp.go#30)

<details>
<summary style="color:#42b983">查看Dial函数源码</summary>

```go
func Dial(addr string) (*Client, error) {
	conn, err := net.Dial("tcp", addr)
	if err != nil {
		return nil, err
	}
	host, _, _ := net.SplitHostPort(addr)
	return NewClient(conn, host)
}
```
</details>

### NewClient
此函可以使用现有与`SMTP`服务器建立的`TCP`连接，返回一个新的`SMTP客户端`

<details>
<summary style="color:#42b983">查看NewClient函数源码</summary>

```go
func NewClient(conn net.Conn, host string) (*Client, error) {
	text := textproto.NewConn(conn)
	_, _, err := text.ReadResponse(220)
	if err != nil {
		text.Close()
		return nil, err
	}
	c := &Client{Text: text, conn: conn, serverName: host, localName: "localhost"}
	_, c.tls = conn.(*tls.Conn)
	return c, nil
}
```
</details>

### (*Client) Auth
Auth使用提供的身份验证机制对客户端进行身份验证，当验证失败会关闭客户端连接，只有支持`AUTH`拓展的`SMTP`服务器才能使用此功能。
<details>
<summary style="color:#42b983">查看(*Client) Auth源码</summary>

```go
func (c *Client) Auth(a Auth) error {
	if err := c.hello(); err != nil {
		return err
	}
	encoding := base64.StdEncoding
	mech, resp, err := a.Start(&ServerInfo{c.serverName, c.tls, c.auth})
	if err != nil {
		c.Quit()
		return err
	}
	resp64 := make([]byte, encoding.EncodedLen(len(resp)))
	encoding.Encode(resp64, resp)
	code, msg64, err := c.cmd(0, strings.TrimSpace(fmt.Sprintf("AUTH %s %s", mech, resp64)))
	for err == nil {
		var msg []byte
		switch code {
		case 334:
			msg, err = encoding.DecodeString(msg64)
		case 235:
			// the last message isn't base64 because it isn't a challenge
			msg = []byte(msg64)
		default:
			err = &textproto.Error{Code: code, Msg: msg64}
		}
		if err == nil {
			resp, err = a.Next(msg, code == 334)
		}
		if err != nil {
			// abort the AUTH
			c.cmd(501, "*")
			c.Quit()
			break
		}
		if resp == nil {
			break
		}
		resp64 = make([]byte, encoding.EncodedLen(len(resp)))
		encoding.Encode(resp64, resp)
		code, msg64, err = c.cmd(0, string(resp64))
	}
	return err
}
```
</details>

### (*Client) Close
此方法用于关闭客户端与`SMTP服务器`的连接
<details>
<summary style="color:#42b983">查看(*Client) Close源码</summary>

```go
func (d *dataCloser) Close() error {
	d.WriteCloser.Close()
	_, _, err := d.c.Text.ReadResponse(250)
	return err
}
```
</details>

###  (*Client) Data
此方法向`SMTP服务器`发出`SMTP`的`DATA`命令，并返回可用于写入邮件头和正文的写入器。**调用者应在调用任何其他方法之前关闭写入器**。在调用`Data`之前，必须先进行一次或多次对`Rcpt`的调用。
返回的`io.WriteCloser`写入数据时应遵守[RFC 822](https://www.rfc-editor.org/rfc/rfc822.html)规范
<details>
<summary style="color:#42b983">查看(*Client) Data源码</summary>

```go
func (c *Client) Data() (io.WriteCloser, error) {
	_, _, err := c.cmd(354, "DATA")
	if err != nil {
		return nil, err
	}
	return &dataCloser{c, c.Text.DotWriter()}, nil
}
```
</details>

### (*Client) Extension
此方法用于查询`SMTP`服务器是否支持传入的拓展(传入的拓展名不区分大小写)，当支持拓展时会返回`true`和对应`SMTP`服务器该`key`下的`value`值字符串，如果不支持就会返回`false`和空字符串。
</br>例如:

```go
// 查询QQ邮箱的`SMTP`服务器是否支持`AUTH`拓展
client,_:=smtp.Dial("smtp.qq.com:25")
b,p:=client.Extension("auth")
log.Println(b)
log.Println(p)

// 输出
true
LOGIN PLAIN
```
<details>
<summary style="color:#42b983">查看(*Client) Extension源码</summary>

```go
func (c *Client) Extension(ext string) (bool, string) {
	if err := c.hello(); err != nil {
		return false, ""
	}
	if c.ext == nil {
		return false, ""
	}
	ext = strings.ToUpper(ext)
	param, ok := c.ext[ext]
	return ok, param
}
```
</details>

### (*Client) Hello


