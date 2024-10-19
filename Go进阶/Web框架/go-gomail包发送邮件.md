# go-gomail介绍

[go-gomail文档](https://godoc.org/gopkg.in/gomail.v2) | [go-gomail/gomail： (github.com)](https://github.com/go-gomail/gomail)

Gomail 是一个简单高效的电子邮件发送包。它经过了良好的测试，并且 记录。

Gomail 只能使用 SMTP 服务器发送电子邮件。但是 API 很灵活，而且 很容易实现使用本地 Postfix 发送电子邮件的其他方法，一个 API 等

Gomail 支持特性：

- 附件
- 嵌入图像
- HTML 和文本模板
- 特殊字符的自动编码
- SSL 和 TLS
- 使用相同的 SMTP 连接发送多封电子邮件

```go
go get gopkg.in/gomail.v2
```

# 简单使用示例

```go
m := gomail.NewMessage()
m.SetHeader("From", "alex@example.com")
m.SetHeader("To", "bob@example.com", "cora@example.com")
m.SetAddressHeader("Cc", "dan@example.com", "Dan")
m.SetHeader("Subject", "Hello!")
m.SetBody("text/html", "Hello <b>Bob</b> and <i>Cora</i>!")
m.Attach("/home/Alex/lolcat.jpg")

d := gomail.NewDialer("smtp.example.com", 587, "user", "123456")

// Send the email to Bob, Cora and Dan.
if err := d.DialAndSend(m); err != nil {
	panic(err)
}
```

# 示例（守护程序）

侦听通道并发送所有传入消息的守护进程。

```go
ch := make(chan *gomail.Message)

go func() {
	d := gomail.NewDialer("smtp.example.com", 587, "user", "123456")

	var s gomail.SendCloser
	var err error
	open := false
	for {
		select {
		case m, ok := <-ch:
			if !ok {
				return
			}
			if !open {
				if s, err = d.Dial(); err != nil {
					panic(err)
				}
				open = true
			}
			if err := gomail.Send(s, m); err != nil {
				log.Print(err)
			}
		// Close the connection to the SMTP server if no email was sent in
		// the last 30 seconds.
		case <-time.After(30 * time.Second):
			if open {
				if err := s.Close(); err != nil {
					panic(err)
				}
				open = false
			}
		}
	}
}()

// Use the channel in your program to send emails.
// Close the channel to stop the mail daemon.
close(ch)
```



# 示例 （Newsletter）

高效地将自定义新闻稿发送给收件人列表。

```go
// The list of recipients.
var list []struct {
	Name    string
	Address string
}

d := gomail.NewDialer("smtp.example.com", 587, "user", "123456")
s, err := d.Dial()
if err != nil {
	panic(err)
}

m := gomail.NewMessage()
for _, r := range list {
	m.SetHeader("From", "no-reply@example.com")
	m.SetAddressHeader("To", r.Address, r.Name)
	m.SetHeader("Subject", "Newsletter #1")
	m.SetBody("text/html", fmt.Sprintf("Hello %s!", r.Name))

	if err := gomail.Send(s, m); err != nil {
		log.Printf("Could not send email to %q: %v", r.Address, err)
	}
	m.Reset()
}
```



# 示例 （NoAuth）

使用本地 SMTP 服务器发送电子邮件。

```go
m := gomail.NewMessage()
m.SetHeader("From", "from@example.com")
m.SetHeader("To", "to@example.com")
m.SetHeader("Subject", "Hello!")
m.SetBody("text/plain", "Hello!")

d := gomail.Dialer{Host: "localhost", Port: 587}
if err := d.DialAndSend(m); err != nil {
	panic(err)
}
```



# 示例 （NoSMTP）

使用 API 或 postfix 发送电子邮件。

```go
m := gomail.NewMessage()
m.SetHeader("From", "from@example.com")
m.SetHeader("To", "to@example.com")
m.SetHeader("Subject", "Hello!")
m.SetBody("text/plain", "Hello!")

s := gomail.SendFunc(func(from string, to []string, msg io.WriterTo) error {
	// Implements you email-sending function, for example by calling
	// an API, or running postfix, etc.
	fmt.Println("From:", from)
	fmt.Println("To:", to)
	return nil
})

if err := gomail.Send(s, m); err != nil {
	panic(err)
}
```

> ```
> Output:
> From: from@example.com
> To: [to@example.com]
> ```