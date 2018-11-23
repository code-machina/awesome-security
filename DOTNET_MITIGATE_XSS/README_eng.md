# .NET Mitigate XSS Threat

Author: gbkim1988@gmail.com

# Summary

To defend against Stored XSS Threat in .NET environment

# Solution 

* multi-Layered Mitigation Strategy
* Whitelisted Html Input Policy
* Ability to handle Malformed HTML, Miss-Structured HTML

## 1. What is "Layered Mitigation Strategy"

Layer means that every processing steps handle html input as an exploit payload which's needed to sanitize. that is, frontend, backend logic must see input as an malformed input. therefore, front-end developers should filter user-input 
