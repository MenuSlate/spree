## Payment Model
* `Payment` tracks payments against `Order`s to decouple logic of processing payments from orders
* Payment `source` indicates how the payment was made
* 8-character identifier is given to each payment when created to prevent gateways from duplication

### States

| State        | Description                                                | Callable with        |
|--------------|------------------------------------------------------------|----------------------|
| `checkout`   | still in checkout                                          |                      |
| `processing` | Temporary state to prevent double submission               | `started_processing` |
| `pending`    | Processed but incomplete (eg. authorized but not captured) | `pend`               |
| `failed`     | Payment rejected (e.g. card was declined)                  | `failure`            |
| `void`       | These payments do NOT count against order total            | `void`               |
| `completed`  | These payments count against order total                   | `complete`           |
