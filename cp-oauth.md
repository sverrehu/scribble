# Using KIP-768 with cp-ansible

Hi! We are currently experimenting with [KIP-768: "Extend
SASL/OAUTHBEARER with Support for
OIDC"](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=186877575) using the Community Edition of Confluent Platform 7.1.1. For deployment, we use cp-ansible.

The only way we found to have SASL protocol `OAUTHBEARER` using
`kafka_broker_custom_listeners`, was to specify `sasl_protocol:
oauth`, like this:

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

This will of course not work, since the requested class does not exist.

We tried setting the property to `null`, but that would end up as an
empty string, resulting in Kafka trying to instantiate a class without a
name.

We also tried setting it to
`org.apache.kafka.common.security.oauthbearer.secured.OAuthBearerLoginCallbackHandler`,
the KIP-768 equivalent of the Confluent class, but that resulted in
demands for several other settings for client-side use, which is, as
we understand it, actually not needed to support client connections.

Now that KIP-768 is out, we suggest either removing the
Confluent-specific overrides from `filters.py`, or, to be backwards
compatible, introduce a `sasl_protocol: oauthbearer` that only sets up
the protocol without filling in the blanks.
