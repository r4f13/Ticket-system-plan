# Structure
# Role
```
enum Role{
  ADMIN
  AGENT
  CUSTOMER
}
```

# API

## auth
### POST auth/register
```ts
{
  username: string,
  password: string,
  role?: Role //"CUSTOMER" by default, only admin can change
}
```
### POST auth/login
```ts
{
  username: string,
  password: string,
}
```

## ticket  (🔒LOGIN REQUIRED)
### GET ticket
- If user is ADMIN, return all ticket
- If user is AGENT, return all assigned ticket
- If user is CUSTOMER, return all requested ticket
### GET ticket/:id
### POST ticket
```ts
{
  subject: string,
  description: string,
  priorityId: number,
  agentId?:number, //ADMIN ONLY, Default: null
  statusId?: number, //ADMIN ONLY, Default: "Unassigned" 
  expiredAt?: timestamp //Default: Date.now()+priority.expiresIn
}
```
### PATCH ticket/:id
```ts
{
  subject?: string, //ADMIN & requester ONLY
  description?: string, //ADMIN & requester ONLY
  priorityId?: number, //ADMIN & requester only
  agentId?:number, //ADMIN & requester ONLY
  statusId?: number, //ADMIN & assigned agent  ONLY
  expiredAt?: timestamp //ADMIN & requester ONLY
}
```
### DELETE ticket/:id
### GET ticket/:ticketId/assign/:agentId
