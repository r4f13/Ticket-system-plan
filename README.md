# Ticketing System

## Database Schema
![](dbschema.png)
```
enum Role{
  CUSTOMER,
  AGENT,
  ADMIN
}
```
```
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
```
{
  username: string,
  password: string
}
```
> User role is "CUSTOMER" by default

**BODY(admin)**
```
{
  username: string,
  password: string,
  role: "CUSTOMER" | "AGENT" | "ADMIN"
}
```
> Admin can set the user role

**RESPONSE**
```
//if username & password is valid
{
  access_token: string
} 
```
#### 2. POST auth/login
**BODY**
```
{
  username: string,
  password: string
}
```
**RESPONSE**
```
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
```
ticket? 
  creatorId = number & 
  status = Status
```

**ROLE BASED RESPONSE**
```
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
```
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
```
{
  subject: string,
  description: string
}
```
**CREATION**
```
{
  subject = body.subject,
  description = body.description,
  status = "unassigned",
  ... 
}
```
> Planned feature: auto assign agent based on some criteria (to be decided)


**BODY (Admin)**
```
{
  subject: string,
  description: string,
  agentId: number
}
```
**CREATION**
```
{
  subject = body.subject,
  description = body.description,
  agentId = body.agentId
  ... 
}
```

#### 4. PATCH ticket/:id
BODY:
```
  title?: string, //ADMIN & REQUESTER ONLY
  description?: string, //ADMIN & REQUESTER ONLY
  status?: Status, //ADMIN & ASSIGNED AGENT ONLY
  agentId?: number, //ADMIN ONLY
  
```
UPDATE:
```
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
```
{
  message: string
}
```
**CREATION**
```
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
```
comment=getCommentById(param.commentId)
if (user.role=="ADMIN" && user.id==comment.senderId){
  //Comment deleted
}else{
  throw Unauthorized
}
```