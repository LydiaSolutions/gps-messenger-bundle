# Google Pub/Sub transport for Symfony Messenger

This bundle lets Symfony applications publish Messenger messages to Google Pub/Sub and receive them through subscriptions. It integrates transport configuration, publishing, polling and acknowledgement with Messenger, and supports ordering keys and Pub/Sub-managed redelivery.

It uses Symfony Messenger, Symfony OptionsResolver and the [Google Cloud Pub/Sub SDK](https://github.com/googleapis/google-cloud-php-pubsub) directly. Unlike the [Enqueue GPS transport](https://github.com/php-enqueue/gps), it requires neither [Enqueue](https://github.com/php-enqueue) nor a [Messenger–Enqueue bridge](https://github.com/sroze/messenger-enqueue-transport#readme).

## Getting started

From the consuming Symfony application, run `composer require petitpress/gps-messenger-bundle --no-scripts` to install the bundle while skipping Composer scripts. Enable the bundle in that application. Consult the [dependency requirements](composer.json) for PHP and package compatibility and the [bundle integration](GpsMessengerBundle.php) for the registration entry point.

Prepare an accessible Pub/Sub topic and a subscription attached to it. The transport publishes and consumes messages; it does not provision these resources.

Make Google configuration available to the PHP process: `GCLOUD_PROJECT` identifies the project, and `GOOGLE_APPLICATION_CREDENTIALS` points to a credentials file when using file-based authentication. For local development with an installed Pub/Sub emulator, configure `PUBSUB_EMULATOR_HOST` (the original local example is `http://localhost:8538`; use the address of your emulator). If Symfony DotEnv supplies these variables, use its `usePutenv()` support so they are accessible through `getenv()`.

Configure a Messenger transport in the consuming application:

```yaml
framework:
    messenger:
        transports:
            gps_transport:
                dsn: 'gps://default/messages'
                options:
                    queue:
                        name: 'messages-subscription'
```

The DSN path selects the publishing topic; `queue.name` selects the receiving subscription. Alternatively, set the topic through `topic.name`. Pull batch size is configurable through `max_messages_pull`, including as a DSN query parameter. Explicit transport options override DSN options; see the [configuration rules](Transport/GpsConfigurationResolver.php).

Route application messages to this transport and use the application's Messenger workers to consume them. Configure serialization consistently with the producer and consumer, then verify delivery and handling against your prepared topic and subscription.

## Delivery behavior

Attach `OrderingKeyStamp` to outgoing envelopes when related messages need a shared ordering key; follow Google's [ordering guidance](https://cloud.google.com/pubsub/docs/publisher#using_ordering_keys).

The receiver retains the native Pub/Sub message for acknowledgement. Rejection releases it for redelivery, while the sender avoids republishing Messenger redeliveries. Configure the subscription retry policy accordingly.

## Development

With development dependencies installed, run `vendor/bin/phpunit` from the repository root. Unit tests cover publishing, ordering keys, suppression of republishing and rejection behavior using mocked Pub/Sub collaborators. The [CI checks](.github/workflows/php.yml) also validate package metadata, coding style and static analysis.

## License

[MIT license](LICENSE), copyright (c) 2018–2020 Petit Press, a.s.
