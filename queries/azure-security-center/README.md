# KQL Queries for Azure Security Center

## Query for List Security Center Secure Score
```
SecurityResources
| where type == 'microsoft.security/securescores/securescorecontrols'
| extend SecureControl = properties.displayName, unhealthy = properties.unhealthyResourceCount, currentscore = properties.score.current, maxscore = properties.score.max, subscriptionId
| project SecureControl , unhealthy, currentscore, maxscore, subscriptionId
```
