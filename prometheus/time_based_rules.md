# Time Based Rules

Here we want to suggest a best practise for writing Prometheus alerting rules
that are time based. The structure should be as follows. At first stands the
metric, which is compared to the time expression on the second position. The
time expression itself makes use of the Prometheus function `time()` on the
first position within the time expression. This is then compared to the
threshold on the second position within the time expression, which we apply to
the alerting rule. The threshold is calculated with ascending time units. To
ease the understanding there should be a comment explaining the alert and the
time comparison.

```
# comment
metric < (time() + (seconds * minutes * hours * days)
```

Consider the following example. The expression should alert when the SSL
certificates of the Ingress Controllers expire in less than 10 days.

```
# Alert when Ingress Controller certificates expire in less than 10 days.
ingress_controller_ssl_expire_time_seconds < (time() + (60 * 60 * 24 * 10)
```
