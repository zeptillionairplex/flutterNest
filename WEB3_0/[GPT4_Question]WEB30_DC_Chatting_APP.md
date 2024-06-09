### Step-by-Step Questions for Building a Decentralized Chat App Using Web 3.0 Technology

Based on your uploaded file, diagram, and requirements, here is a structured sequence of questions to guide the development of a decentralized chat app using Web 3.0 technology:

```
+-------------------+                +-------------------+
|                   |                |                   |
|   Smart Phone A   |                |   Smart Phone B   |
|                   |                |                   |
| +--------------+  |                | +--------------+  |
| | IPFS 노드    |  |                | | IPFS 노드    |  |
| +--------------+  |                | +--------------+  |
|                   |                |                   |
| +--------------+  |                | +--------------+  |
| | PubSub       |  |                | | PubSub       |  |
| +--------------+  |                | +--------------+  |
|                   |                |                   |
| +--------------+  |                | +--------------+  |
| | WebRTC/Libp2p|  |                | | WebRTC/Libp2p|  |
| +--------------+  |                | +--------------+  |
|                   |                |                   |
| +--------------+  |                | +--------------+  |
| | DID 인증     |  |                | | DID 인증     |  |
| +--------------+  |                | +--------------+  |
|                   |                |                   |
+-------------------+                +-------------------+

+---------------+                +---------------+
|               |                |               |
| NAT 방화벽   |                | NAT 방화벽   |
|               |                |               |
+---------------+                +---------------+

+-----------------------------------------------+
|         STUN/TURN 서버 (NAT 트래버설)         |
+-----------------------------------------------+
```

#### Step 1: Project Overview and Initial Setup
**Question 1**:
"As a highly experienced full-stack developer with 100 years of experience, I want to create a decentralized chat app using Web 3.0 technology without a traditional server. The frontend will be an Android Native App in Kotlin, and the backend will use Node.js not Nest.js. Please provide the overall file tree structure of this project and explain the role of each file, including the frontend, backend, and configurations."

#### Step 2: Frontend Basic Setup
**Question 2**:
"Set up a basic Android Native App in Kotlin project for the frontend. Create an initial screen that displays 'Welcome to Decentralized Chat'. Provide detailed comments explaining the role and function of each part of the code. Include instructions on configuring the app to use the internet and add the necessary permission code to the Android build settings."

#### Step 3: Backend Basic Setup
**Question 3**:
"Set up a basic Node.js not Nest.js project for the backend. Include the necessary configurations for handling WebRTC, PubSub, and IPFS connections. Provide detailed comments explaining the role and configuration of each part of the code. Also, include Dockerfile and docker-compose.yml files to run the backend services in Docker."

#### Step 4: Implementing IPFS and PubSub
**Question 4**:
"Integrate IPFS and PubSub into the frontend and backend. Ensure that both smartphones (A and B) can communicate via IPFS nodes and PubSub. Provide detailed comments explaining how IPFS nodes are set up and how PubSub messaging works. Include code examples for both the frontend and backend."

#### Step 5: Implementing WebRTC/Libp2p
**Question 5**:
"Integrate WebRTC/Libp2p into the frontend and backend for direct peer-to-peer communication. Ensure that both smartphones (A and B) can establish a direct connection using WebRTC/Libp2p. Provide detailed comments explaining the role and configuration of each part of the code. Include code examples for both the frontend and backend."

#### Step 6: Implementing DID Authentication
**Question 6**:
"Implement Decentralized Identifier (DID) authentication for both the frontend and backend. Ensure that users can authenticate themselves using DIDs and that the authentication process is secure. Provide detailed comments explaining the role and function of each part of the code. Include code examples for both the frontend and backend."

#### Step 7: NAT Traversal with STUN/TURN Servers
**Question 7**:
"Set up STUN/TURN servers for NAT traversal to ensure that peer-to-peer connections can be established even behind NAT firewalls. Provide detailed comments explaining the role and configuration of each part of the code. Include code examples for both the frontend and backend."

#### Step 8: Encryption and Data Security
**Question 8**:
"Apply encryption to the data exchanged between peers over the internet. Ensure that all communication is secure. Provide detailed comments explaining the encryption methods used and the role of each part of the code. Include code examples for both the frontend and backend."

#### Step 9: Using DTO, Provider, Repository Design Pattern
**Question 9**:
"Using the existing frontend and backend code developed in the previous steps, apply the DTO, Provider, and Repository design patterns. Integrate these patterns into the current project structure and update the existing files accordingly. Provide detailed comments explaining the role and implementation method of each pattern, and ensure that the updated code maintains the current functionality."

#### Step 10: Integrating Docker
**Question 10**:
"Integrate Docker into the project for easy deployment. Create Dockerfile and docker-compose.yml files for the backend. Provide detailed comments explaining the role and configuration of each part of the Docker setup. Include instructions on how to start the services using Docker."

#### Step 11: Final Integration and Testing
**Question 11**:
"Integrate all the components developed in the previous steps into a cohesive platform. Test the entire system thoroughly, identify potential issues, and provide methods to resolve them. Provide detailed comments and explanations on how to achieve this."

#### Step 12: Comprehensive Project Manual
**Question 12**:
"Write a comprehensive manual that explains how to follow this project from start to finish. Include the necessary commands and explanations for each step, and provide detailed instructions on deploying the application using Docker. Ensure the manual is detailed enough for someone to replicate the project with a simple copy and paste of the code."

#### Step 13: Conclusion and Verification
**Question 13**:
"After completing the above steps, please indicate 'Answer Done' and provide a summary of the entire project with any additional tips or best practices for maintaining and scaling the decentralized chat app."
