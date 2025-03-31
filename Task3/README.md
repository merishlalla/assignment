## Multiple Production Instance Monitoring

### First thoughts:

How would we monitor a single instance? If we can it for one instance then we can likely do it for multiple but how?

These are the questions I'd ask myself or the team:
- How are the metric stored configured? 
    - Are all metrics for every instance stored in the same manner? (Example: There's no identifier. What this could mean is we'd see the same metrics or logs for Cape Town and Munich at the same time but we wouldn't know the source from only logs)(Or we could if there was a flag or identifier set but how easy would this be?).
    - Does the metrics store scale?
    - How easy is it to set up this disparity?
    - With this disparity tracking set up, how do we know the data recieved is accurate? (Example person A is experiencing a timeout but we don't know if it's Munich or Hamburg.)
    - If it's accurate: how do we know it's actionable?
    - How can we "trust but verify"? (Basically make absolute sure the alert we handle is correct)
    - In the event the location is set up:
        - Are we working with usable data?
            - Usuable data could mean the following format:
                {
                    username: string,
                    location: string,
                    time: dataTime,
                    Emergency: boolean,
                    ContactInfo: string,
                    doctorsAvailableNow: [{doctorA}, {doctorB}, {doctorC}...{doctorZ}] || None
                    doctorLanguage: [{German}, {English}]
                    doctorFutureAvailability: [{dateTime}, {dateTime}, {dateTime}] #Could also be a range
                    TimeZone: [{DaylightSavings: boolean}]
                }

The following checks need to be done as data and monitoring needs to be actionable. This stems from the Google SRE belief that all alerts should be actionable. Because actionable has been said, we also need to define this.

Actionable definition can also be based on SLA:
- Doctors work from 8am-5pm.
- Emergency doctors work from 7pm until 3am. (Hypothetically)
- Medical interns cover the in between periods. (5pm-7m and 3am-8am)
    - Or there would be no interns and during these periods, a person goes straight to the Emergency Room. 

Now that we have example SLAs defined, we can define SLO:
- During operational hours: we expect a maximum failure rate of 3%. As we expect this, we can set a KPI (key performance indicator) and maybe use this as a threshold limit or threshold level. 
- We have now hit the threshold and we know it's actionable so what is the best course of action?
    - Look at data (logs, metrics, errors, customer issues or issues, time occured)
    - This way we also have more actionable data.
        - Example we experience an outage at midnight. During this instance, the platform should be quiet and we have more time to fix it prior to the rush of the next day. 
        - What about hospitals? For this use case, we need to jump in immediately. (I know currently this use case is not in scope but it doesn't mean it might never be).
        - Additionally if patient x is feeling ill at 4am, can we assist at this hour? The user then might not be able to book an appointment for their nearest doctor for first thing in the morning. 
            - In the event this is during something like an event where 300 out of 1500 people get food poisoning, how can our systems handle this? (Regardless of time)

Based on the above idea is an overview of granularity of the problem. The `solution` could look like:

- How can we prevent the instance from remaining down? What are the error response codes we've detected? Do we have a scale issue? (example we scale down at 2am-6am but now there's a large load). Can we scale up? Is autoscaling broken? 

## The above outline a few instances of issues I can think of but monitoring side (and to answer the questions):

- Set up relevent metrics
- Define a metrics retention period. (Example: Metrics from 3 years ago won't help us now)
- Ensure metrics can be checked by a certain criteria (example Munich or Hamburg).
- Analyze the metrics. 
    - We see 10 requests fail but is this actionable is we have 3000 succeeding. 
    - Do we have a retry limit (Example after 3 attempts do we allow a forth? Do we allow endless? Can we open ourself up to a DDoS attack in this way?)
    - Be willing to update as we go. 
        - The user volume is likely to increase over time. This is how we can access the thresholds. (If 200 out of 200 000 are failing, is it probable that it's us or the user? What if someone just got bored and left their device and that's why a session wasn't completed on comparison to the user tried to book an appointment and couldn't. (408 vs 503 or 500))

## Tech Stack to achieve this:

- Elastic search 
- Logstash or equivalent
- Ops Genie/Pager Duty/Alerting system
- Kibana
- Stern (For kubernetes. Also in the event Kibana is not operational for reason x.)
- Terminal commands ("grep" or something like "grep C -5 <message>")

## Personal style of metrics and monitoring:

Use case: When we have a service, there would be multiple endpoints or this service handles the tasks of A,B,C.

We now have detected an issue however the automated alert only specifies "Error on A". (Side note: Alert messages should be descriptive)

- Check if it is a problem.
    - Yes it's a problem: 
        - Check if this related to a deploy or any other system change?
        - What's the impact?
        - How urgent is it?
        - If all of these can be answered as "no", "none", "not really":
            - Reconfigure the alert. All alerts should be actionable.
        - If yes:
            - Can I handle this myself? 
            - Do I know enough? 
            - Let's get started!

# Final thoughts:

Metrics and Monitoring wise:

- If Service A is new, add metrics to EVERYTHING. Then define the alerting on this. It's better to have and not need in comparison to need and not have.
    - This will only be a predefined period of time. My logic is: "If component x doesn't break now, it likely won't break unless we change it a lot and we know what or how we're changing it."

- If Service B is older: Refine what's actionable. Configure and reconfgure monitoring and alerts if necessary. 
    - Maybe encourage a code clean up? Yes it's working but doesn't mean it can't be improved. Example if this was built 10 years ago and never touched again, likely we can update it to the latest code version (example Python 2.6 -> 3 -> 3.latest [Gradual upgrade to assess and minimize the impact]. This can also be mitigated with an immediate roll back if things go sideways.)

