# Ticketing System

## Database Schema
![](dbschema.png)
```ts
enum Role{
  CUSTOMER,
  AGENT,
  ADMIN
}
```
```ts
enum Status{
  UNASSIGNED, 
  WAITING,
  ONPROGRESS,
  RESOLVED,
  CANCELLED
}
```

## Module
1. Auth: Berkaitan dengan autentikasi user seperti login dan register
2. User: Berkaitan dengan pengambilan informasi user
3. Ticket: Berkaitan dengan pembuatan, pembaruan, dan penghapusan tiket
4. Comment: Berkaitan dengan pengiriman dan penghapusan komentar

## Usage
### 1. Auth
#### 1. POST auth/register
**BODY(non-admin)**
```ts
{
  username: string,
  password: string
}
```
> User role is "CUSTOMER" by default

**BODY(admin)**
```ts
{
  username: string,
  password: string,
  role: "CUSTOMER" | "AGENT" | "ADMIN"
}
```
> Admin can set the user role

**RESPONSE**
```ts
//if username & password is valid
{
  access_token: string
} 
```
#### 2. POST auth/login
**BODY**
```ts
{
  username: string,
  password: string
}
```
**RESPONSE**
```ts
//if username & password is correct
{
  access_token: string
} 
```
### 2. User

#### 1. GET user (ADMIN ONLY)
Return all user

#### 2. GET user/customer (ADMIN ONLY)
Return all customer

#### 3. GET user/admin (ADMIN ONLY)
Return all admin

#### 4. GET user/agent
Return all agent

### 3. Ticket

#### 1. GET ticket
**QUERY**
```url
ticket? 
  creatorId = number & 
  status = Status
```

**ROLE BASED RESPONSE**
```ts
if(user.role=="ADMIN"){
  return //get all tickets (+query filter)
}
else if(user.role=="AGENT"){
  return //get all tickets where assignedId=user.id or status="unassigned" (+query filter)
}
else if(user.role=="customer"){
  return //get all tickets where requesterId=user.id (+query filter)
}
else{
  throw Unauthorized
}
```

#### 2. GET ticket/:id
**RESPONSE**
```ts
//if user.role=="ADMIN" || user.id==ticket.requesterId || user.id==ticket.agentId
{
  subject: string,
  description: string,
  status: Status,
  comment: [{
    senderId: number, 
    message: string
  }]
}

//else -> Unauthorized
```

#### 3. POST ticket
**BODY (Non-admin)**
```ts
{
  subject: string,
  description: string
}
```
**CREATION**
```ts
{
  subject = body.subject,
  description = body.description,
  status = "unassigned",
  ... 
}
```
> Planned feature: auto assign agent based on some criteria (to be decided)


**BODY (Admin)**
```ts
{
  subject: string,
  description: string,
  agentId: number
}
```
**CREATION**
```ts
{
  subject = body.subject,
  description = body.description,
  agentId = body.agentId
  ... 
}
```

#### 4. PATCH ticket/:id
BODY:
```ts
{
  title?: string, //ADMIN & REQUESTER ONLY
  description?: string, //ADMIN & REQUESTER ONLY
  status?: Status, //ADMIN & ASSIGNED AGENT ONLY
  agentId?: number, //ADMIN ONLY
}
```
UPDATE:
```ts
//ticket=getTicketById(param.id)
//if user.role=="ADMIN" || ticket.requesterId
//update where ticket.id=param.id
{
 ...body
}

//else -> Unauthorized
```

### 4. Comment
#### 1. POST comment/:ticketId
**BODY**
```ts
{
  message: string
}
```
**CREATION**
```ts
//ticket=getTicketById(param.ticketId)
//if user.id==ticket.requesterId || user.id==ticket.agentId
{
  message = body.message,
  senderId = user.id,
  ticketId = ticketId
}

//else -> Unauthorized
```

#### 2. DELETE comment/:commentId
```ts
comment=getCommentById(param.commentId)
if (user.role=="ADMIN" && user.id==comment.senderId){
  //Comment deleted
}else{
  throw Unauthorized
}
```
