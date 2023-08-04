<div align="center">
    <p>
        <img src="./docs/static/logo.png" width="200" height="200"/>
        <h1 align="center">CalendarSync</h1>
        <b>Stateless CLI tool to sync calendars across different calendaring systems.</b>
    </p>
</div>

# Motivation

As consultants you may need to use multiple calendars (2-n). Additionally you
need to keep up with all existing appointments in each of your calendars when
you want to make new appointments. This means you have to check each calendar on
its own. What we wanted to achieve is a single overview over all events in each
of the calendars. Preferrably in your primary calendar.

There are some commercial / freemium solutions for this
([reclaim.ai](https://reclaim.ai/),
[SyncThemCalendars](https://syncthemcalendars.com/)), but their privacy policy
is unclear. Calendar data is not only highly interesting personal data (who
participates in which appointment and when?) but also highly interesting from an
industrial espionage/targeted advertising perspective. The two third party
providers get to see the content of the calendar events. In good appointments,
there is a lot of secret and relevant company data in the appointment agenda.

To keep track of all the events, we created `CalendarSync`, which allows the
syncing of events without breaking data protection laws and without exposing
data to a third party.

# How to use

Download the newest
[release](https://github.com/inovex/CalendarSync/releases)
for your platform, create a modified `sync.yaml` file based on the content of
  the `./example.sync.yaml` file. Finally, start the app using `./calendarsync
  --config sync.yaml --storage-encryption-key <YourSecretPassword>` and follow
  the instructions in the output.

The app will create a file in the execution folder called `auth-storage.yml`. In
this file the OAuth2 Credentials will be saved encrypted by your
`storage-encryption-key`.

# Configuration

The CalendarSync config file consists of `four` building blocks:

- `sync` - Controls the timeframe to be synced
- `source` - Controls the source calendar to be synced from
- `sink`- Controls the sink (target) calendar where the events from the source
  calendar are written to
- `transformations` - Controls the transformers applied to the events before
  syncing

## Sync

Should be self-explanatory. Configures the timeframe where to sync events. The
currently only implemented identifiers are `MonthStart` and `MonthEnd`.

```yaml
sync: 
  start: 
    identifier: MonthStart # 1st of the current month 
    offset: -1 # MonthStart -1 month (beginning of last month) 
  end: 
    identifier: MonthEnd # last day of the current month 
    offset: +1 # MonthEnd +1 month (end of next month)
```

## Source

Example:

```yaml
source: 
  adapter: 
    type: "outlook_http" 
    calendar: "[base64-formatstring here]" 
    oAuth: 
      clientId: "[UUID-format string here]" 
      tenantId: "[UUID-format string here]" 
```

Configures the Source Adapter, for the adapter configuration, check the
documentation [here](./docs/adapters.md).

### Available Source Adapters

- Google
- Outlook
- [ZEP](https://www.zep.de/en/)

## Sink

Example:

```yaml
sink:
  adapter: 
    type: google 
    calendar: "target-calendar@group.calendar.google.com" 
    oAuth: 
      clientId: "[google-oAuth-client-id]"
      clientKey: "[google-oAuth-client-key]" 
```

Configures the Sink Adapter, for the adapter configuration, check the
documentation [here](./docs/adapters.md).

### Available Sink Adapters

- Google
- Outlook

## Transformers

Basically, only the time is synced. By means of transformers one can sync
individual further data. Some transformers allow for further configuration using
an additional `config` block, such as the `ReplaceTitle` transformer. Below is a
list of all transformers available:

```yaml
transformations:
  - name: KeepDescription
  - name: KeepLocation
  - name: KeepReminders
  - name: KeepTitle
  - name: PrefixTitle 
    config: 
      Prefix: "[Sync] "
  - name: ReplaceTitle 
    config: 
      NewTitle: "[synchronized appointment]" # Do not use KeepAttendees when the Outlook Adapter is used as a sink. There is no way to suppress mail invitations
  - name: KeepAttendees 
    config: 
      UseEmailAsDisplayName: true 
```

The transformers are applied in a specific order. The order is defined here:
[`internal/sync/transformer.go`](./internal/sync/transformer.go)

# Cleaning Up

You just synced a lot of events in your calendar and decide you want to use a
separate calendar for this? Or you simply want to remove all the synced events
from your calendar?

Use the `--clean` flag to get rid of all the unwanted events. (We leave your
events which were't synced with CalendarSync alone! :) )

# Trademarks

GOOGLE is a trademark of GOOGLE INC. OUTLOOK is a trademark of Microsoft
Corporation

# Relevant RFCs and Links

[RFC 5545](https://datatracker.ietf.org/doc/html/rfc5545)  Internet Calendaring
and Scheduling Core Object Specification (iCalendar) is used in the google
calendar API to denote recurrence patterns. CalDav [RFC
4791](https://datatracker.ietf.org/doc/html/rfc4791) uses the dateformat
specified in RFC 5545.

# License

MIT
