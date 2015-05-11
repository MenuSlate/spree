# Security Alerts
* Spree periodically checks for important security and release announcements and you will see 
them on the administration console pages
* Alerts may be dismissed after you have read them. 
* automatic checking can be disabled under (Configuration -> General Settings) or setting
`Spree::Config[:check_for_alerts]` to `false`. 
* Some configuration information is included when checking so alerts can be customized

Here is an example of the configuration information included in the alert request:

```json
{
  "name": "Spree Demo Site",
  "rails_version": "3.1.1",
  "version": "0.70.1",
  "rails_env": "production",
  "host": "www.spreecommerce.com"
}
```
