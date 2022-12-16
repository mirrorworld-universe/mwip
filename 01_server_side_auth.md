# [MWIP - 1] Server side authentication proposal

### Summary

- Provide a way for developers to use SDK and APIs with backend service
- Make it easier to get started using SDK and APIs 

## Generate Bearer Token on the project

- Provide an option to generate bearer token in the developer dashboard, next to the API key
- Developer can then use that bearer token when initiating SDK inside the service or as Authorization header when using API
- Bearer token should be scoped, so it can only work with the API key from that particular project

