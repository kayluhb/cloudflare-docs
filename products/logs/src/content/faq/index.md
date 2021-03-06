---
title: FAQs
alwaysopen: true
weight: 130
---

- **General**
  - [Once a request has passed through the Cloudflare network, how soon are the logs available?](#once-a-request-has-passed-through-the-cloudflare-network-how-soon-are-the-logs-available)
  - [How long are logs retained?](#how-long-are-logs-retained)
  - [Are logs available for customers who are not on an Enterprise plan?](#are-logs-available-for-customers-who-are-not-on-an-enterprise-plan)
  - [When pulling or pushing logs, I occasionally come across a time period with no data, even though I’m sure my domain received requests at that time. Is this normal?](#no-data)
  - [Can I receive logs in a format other than JSON?](#can-i-receive-logs-in-a-format-other-than-json)
- **Logpull API**
  - [I’m asking for logs for the time window of 16:10-16:13. However, the timestamps in the logs show requests that are before this time period. Why does that happen?](#before-time-period)
- **Logpush**
  - [What happens if my cloud storage destination is temporarily unavailable?](#what-happens-if-my-cloud-storage-destination-is-temporarily-unavailable)
  - [Can I adjust how often logs are pushed?](#can-i-adjust-how-often-logs-are-pushed)
  - [My job was accidentally turned off, and I didn’t receive my logs for a certain time period. Can they still be pushed to me?](#job-turned-off)

---

### General FAQ

##### <a id="once-a-request-has-passed-through-the-cloudflare-network-how-soon-are-the-logs-available"></a>Once a request has passed through the Cloudflare network, how soon are the logs available?

Logs become available in approximately 1 to 5 minutes.

In the best case, logs take about 1 minute to process, and so we require that calls to the **Logpull API** be for time periods of at least 1 minute in the past. For example, if it’s 9:43 now, you can ask for logs processed between 9:41 and 9:42. The response will include logs for requests that passed through our network between 9:41 and 9:42 and potentially earlier. It’s normal for our processing to take between 3 and 4 minutes, so when you ask for that same time period, you may also see logs of requests that passed through our network at 9:39 or earlier.

When using **Logpush**, logs are pushed in 5-minute batches, with at least a 5 to 10 minute delay. For example, if it’s 10:10, you’ll receive a file of logs that were processed between 10 and 10:05, which may include requests that passed through our network as late as 10:04 (assuming the best case processing delay).

These timings are only a guideline, not a guarantee, and may depend on network conditions, the request volume for your domain, and other factors. Although we try to get the logs to you as fast as possible, we prioritize not losing log data over speed. On rare occasions, you may see a longer delay. In this case, you don’t need to take any action--the logs will be available as soon as they’re processed.

##### <a id="how-long-are-logs-retained"></a>How long are logs retained?

Cloudflare makes logs available for at least 3 days and up to 7 days. If you need your logs for a longer time period, download and store them locally.

##### <a id="are-logs-available-for-customers-who-are-not-on-an-enterprise-plan"></a>Are logs available for customers who are not on an Enterprise plan?

Not yet, but we’re planning to make them available to other customer plans in the future.

##### <a id="no-data"></a>When pulling or pushing logs, I occasionally come across a time period with no data, even though I’m sure my domain received requests at that time. Is this normal?

Yes, this is normal. The time period for which you pull or receive logs is based on our processing time, not the time the requests passed through our network. If you receive an empty response, it does not mean there were no requests during that time period. It just means we did not process any logs for your domain during that time.

##### <a id="can-i-receive-logs-in-a-format-other-than-json"></a>Can I receive logs in a format other than JSON?

Currently not. Talk to your account manager or Cloudflare Support if you’re interested in other formats and we’ll consider them for the future.

---

### Logpull API FAQ

##### <a id="before-time-period"></a>I’m asking for logs for the time window of 16:10-16:13. However, the timestamps in the logs show requests that are before this time period. Why does that happen?

When you make a call for the time period of 16:10-16:13, you're actually asking for the logs that were received and processed by our system during that time (hence the endpoint name, _logs/received_). the _received_ time is the time the logs are written to disk. There is some delay between the time the request hits the Cloudflare edge and the time it is received and processed. The _request time_ is what you see in the log itself: _EdgeStartTimestamp_ and _EdgeEndTimestamp_ tell you when the edge started and stopped processing the request.

The advantage of basing the responses on the _time received_ rather than the request or edge time is not needing to worry about late-arriving logs. As long as you're calling our API for continuous time segments, you will always get all of your logs and will never have to make the same call twice. If we based the response on request time, you could never be sure that all the logs for that request time had been processed.

---

### Logpush FAQ

##### <a id="what-happens-if-my-cloud-storage-destination-is-temporarily-unavailable"></a>What happens if my cloud storage destination is temporarily unavailable?

**Logpush** is designed to retry in case of errors. If your destination is temporarily unavailable, we’ll keep trying until it’s online again and the logs are received. We’ll also automatically catch up, so that you don’t miss any logs. However, if we persistently receive errors from your destination for 7 days, we’ll take that as a sign that it’s permanently unavailable and disable your push job. It can always be re-enabled later.

##### <a id="can-i-adjust-how-often-logs-are-pushed"></a>Can I adjust how often logs are pushed?

Currently not. We may offer faster speeds or more flexibility in the future. For now, if you’d like to download logs at a different frequency, you can use the **Logpull API**, which allows you to specify the time period.

##### <a id="job-turned-off"></a>My job was accidentally turned off, and I didn’t receive my logs for a certain time period. Can they still be pushed to me?

No, **Logpush** only pushes the logs once as they become available, and is unable to backfill. However, the logs are stored for a period of at least 72 hours and can be downloaded using the **Logpull API**.
