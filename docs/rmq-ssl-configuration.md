RabbitMQ and SSL configuration
---

Configuring SSL on RabbitMQ is fairly simple, if you follow the SSL guide posted on http://www.rabbitmq.com/ssl.html

Important points:

- Ensure you generate the certificates using OpenSSL
- Generate the certificates with the proper hostname.
- Ensure the rabbitmq.config file is created in the right location. (/usr/local/etc/rabbitmq)
- If you need just SSL, copy the following configuration as is into the rabbit.config file
- The configuration allows clients to connects using certificates, or without using certificates to the SSL port `5671`.


`rabbitmq.config` file contents:


```
[
  {rabbit, [
     {ssl_listeners, [5671] },
     {ssl_options, [{cacertfile,"/[location to the cert]/testca/cacert.pem"},
                    {certfile,"/[location to the cert]/server/cert.pem"},
                    {keyfile,"/[location to the cert]/server/key.pem"},
                    {verify,verify_peer},
                    {fail_if_no_peer_cert,false}]}
   ]}
].
```

Installing RabbitMQ Server on Mac using `homebrew`

From the terminal prompt:

`$ brew install rabbitmq-server`

Important file locations:

- Logs: `/usr/local/var/log/rabbitmq`
- Config: `/usr/local/etc/rabbitmq`
- Mnesia Database: `/usr/local/var/lib/rabbitmq/mnesia`
- Erlang cookie: `/.erlang.cookie`
