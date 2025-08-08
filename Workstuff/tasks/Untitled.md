This epic tracks the implementation of a centralized login solution for the main website and other frontend repositories. The goal is to create a unified authentication system, using either a micro-frontend or a shared package, to manage login, logout, session handling, and token management. This solution will ensure compatibility with React (client-side) and Next.js (server-side) environments, supporting both stateless and stateful session handling as required by specific use cases.

Additionally, this epic includes drafting the Technical Requirements Document (TRD) to discuss and resolve open questions about dependencies, session management strategies, token handling, and implementation approaches.

---

### **Goals of the Epic**

1. Develop a centralized authentication system for consistent login/logout management, across multiple applications.
2. our soln should have minimum interruptions possible to the existing system. (dev effort, migration effort, backend effort)

---

## Functional requirements

- system should support
	- email - p1
	- phone - p1
	- should extend to SSO as well in the future (okta, and other SSO providers) - p2
- backward compatibility - p1
	- must have: should be compatible with the UMS token as well auth0 token.
- social auth - p2
- Ensure that minimum interruptions/dev effort is required for the migration from old login to the new omni login.
- should easily support integration with multiple web ui apps.
- good to have: should be able sharable with external parties. (should be configurable)
- good to have UI: should be configurable.
- good to have: stateless and stateful token both should be managed
- platform level permissions:
	- part of the configuration. if A and B service both use this login, we should be able to ...

## Non functional requirements
- performance: load times should be minimum. 
- Security:
	- passing of access tokens, CSRF etc should be in consideration. 
- UI: should be portable, modular. 

### **Key Deliverables**

1. **Technical Requirements Document (TRD):**
    
    - Outline the micro-frontend vs package approach.
        
    - Specify dependencies, integration points, and workflows for session handling and login mechanisms.
        
2. **Centralized Authentication Solution:**
    
    - A reusable micro-frontend or package for login/logout.
        
    - Seamless integration with the main website and other repositories.
        
3. **Session Management:**
    
    - Stateless sessions using tokens (JWT).
        
    - Stateful sessions using server-managed storage (e.g., cookies or Redis).
        
4. **Frontend and Backend Dependencies:**
    
    - Identify all locations in the UI relying on auth0 sessions or tokens.
        
    - Address backend dependencies for session validation and session storage.
        
    - Evaluate WiFi and Printer dependency on auth0
        
5. **Roles and Permissions Management:**
    
    - Define a strategy to handle user details, roles, and permissions across applications.
        

---

### **Requirements for Client-Side and Server-Side Login Handling**

#### **Client-Side Login Handling**

1. **Token-Based Authentication:**
    
    - Implement storage of access and refresh tokens securely on the client-side (e.g., HTTP-only cookies or secure local storage).
        
    - Handle token expiration and automatic token refresh workflows.
        
2. **Session Management:**
    
    - Ensure compatibility with stateless (JWT) sessions for REST APIs or GraphQL.
        
    - Provide utilities for validating user sessions and redirecting unauthorized users.
        
3. **User Session Management in React:**
    
    - Create a session context to manage user state globally.
        
    - Update session state on login, logout, and token refresh events.
        
4. **Security Considerations:**
    
    - Use CSRF protection for cookie-based token storage.
        
    - Secure API calls with access tokens in authorization headers.
        
    - Not exposing the sensitive data to other
        

---

#### **Server-Side Login Handling (Next.js)**

1. **Authentication on the Server:**
    
    - Use Next.js API routes or middleware to handle login, logout, and session validation.
        
    - Validate tokens (e.g., JWT) and issue new tokens upon refresh requests.
        
2. **Stateful Session Management:**
    
    - Store sessions securely in a server-managed store (e.g., Redis or database).
        
    - Use HTTP-only cookies to manage session IDs.
        
3. **Stateless Session Handling:**
    
    - Verify JWTs on each request without maintaining server-side session storage.
        
    - Integrate token expiry and claims-based authentication for resource access.
        
4. **User Authentication in Next.js Pages:**
    
    - Use getServerSideProps to validate user sessions and fetch authenticated data.
        
    - Redirect unauthenticated users to the login page.
        

---

### **Requirements for Stateless vs Stateful Sessions**

1. **Stateless Sessions (JWT):**
    
    - Use JWTs for access and refresh tokens.
        
    - Validate JWTs on each API request without storing session data on the server.
        
    - Handle token expiry and rotation securely.
        
2. **Stateful Sessions:**
    
    - Store session data in a backend store (e.g., Redis) with a unique session ID.
        
    - Use HTTP-only cookies for session ID management.
        
    - Provide an efficient mechanism to invalidate sessions upon logout.
        
3. **Hybrid Approach (Optional):**
    
    - Use stateless sessions (JWT) for most API interactions.
        
    - Fall back to stateful sessions for specific use cases (e.g., enhanced security or persistent logins).
        

---

### **Tasks for the Epic**

#### **1. Evaluate Micro-Frontend vs Package Approach**

- Draft a TRD to document the pros and cons of each approach.
    
- Develop a proof-of-concept for both approaches.
    

#### **2. Implement Client-Side Session Management**

- Develop utilities to manage access and refresh tokens in React.
    
- Create a session context to manage user state globally.
    
- Implement automatic token refresh and session expiration handling.
    

#### **3. Implement Server-Side Session Management**

- Configure Next.js middleware for session validation and token management.
    
- Integrate stateful session storage (Redis or database) for server-managed sessions.
    
- Provide APIs for login, logout, and token refresh workflows.
    

#### **4. Token Handling and Security**

- Design workflows for managing UMS tokens, OKTA tokens, and social login tokens.
    
- Secure token storage and transfer using best practices (e.g., HTTP-only cookies, HTTPS).
    
- Implement CSRF protection and secure token rotation.
    

#### **5. Integration with Frontend Repositories**

- Build reusable login/logout functionality as a micro-frontend or package.
    
- Integrate the solution into the main website and other repositories.
    

#### **6. Handle Roles, Permissions, and User Details**

- Define a centralized strategy for managing roles, permissions, and user details.
    
- Ensure compatibility with existing role-based access control (RBAC) systems.
    

#### **7. Identify Auth0 dependencies and Solutioning**

- What all locations in UI do we depend on auth0 token
    
- BE dependencies
    

#### **8. List down all the dependencies that needs to be resolved.**

- List down dependencies like design dependencies, global dependencies and existing architecture restrictions, etc.,
    
- Migrations that needs to done.
    

---

### **Acceptance Criteria**

- A reusable centralized authentication system (micro-frontend or package) which is built on top of our new UMS where-ever possible is developed and integrated into the main website and other repositories.
    
- Client-side and server-side session handling are implemented securely.
    
- Stateless (JWT) and stateful session handling are supported as required.
    
- Clear guidelines and utilities for token handling and security are provided.
    
- User roles, permissions, and details are managed consistently across applications.
    
- Security shouldn’t be comprimised.