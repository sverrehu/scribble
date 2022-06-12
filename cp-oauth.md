# Using KIP-768 with cp-ansible

Hi! We are currently experimenting with [KIP-768: "Extend
SASL/OAUTHBEARER with Support for
OIDC"](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=186877575) using the Community Edition of Confluent Platform 7.1.1. For deployment, we use cp-ansible.

The only way I we found to have SASL protocol `OAUTHBEARER` using
`kafka_broker_custom_listeners`, appears to be to specify
`sasl_protocol: oauth`, like this:

```yaml
kafka_broker_custom_listeners:
  oauth:
    name: OAUTH
    port: 9095
    sasl_protocol: oauth
    ssl_enabled: true
    ssl_mutual_auth_enabled: false
```

We want to enable clients to connect using OAuth, without using OAuth
between brokers, so we also specify this:

```yaml
kafka_broker_custom_properties:
  listener.name.oauth.oauthbearer.sasl.jaas.config: org.apache.kafka.common.security.oauthbearer.OAuthBearerLoginModule required;
  listener.name.oauth.oauthbearer.sasl.server.callback.handler.class: org.apache.kafka.common.security.oauthbearer.secured.OAuthBearerValidatorCallbackHandler
  listener.name.oauth.oauthbearer.sasl.oauthbearer.jwks.endpoint.url: ...
```

However, due to [code in `filters.py` from
2020](https://github.com/confluentinc/cp-ansible/commit/83d845d4850a44e5086532af3f4128981b839619),
long before KIP-768, some properties are populated with
Confluent-proprietary classes, so we end up with this in `server.properties`:

```properties
listener.name.oauth.oauthbearer.sasl.login.callback.handler.class=io.confluent.kafka.server.plugins.auth.token.TokenBearerServerLoginCallbackHandler
```
