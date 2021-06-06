## What Is System Design / System Architecture?

Simply, I like to define system design as **the trade-off decisions you make to build a well-running scalable system**.

There isn't a perfect nor the best system architecture out there that you can apply for all your projects it all depends on the system's requirements you are trying to build. for example, building chat applications is nothing like building e-commerce apps both have different functional and non-functional requirements.

## What Are The Main System Architecture Components?

A simple system architecture can be something like this

![/images/sys-design-basics/3-tier.png](/images/sys-design-basics/3-tier.png)

**Client (Presentation tier)**: it is the user interface and communication layer of the application, where the end-user interacts with the application.

presentation tiers are usually developed using HTML, CSS, and JavaScript. Desktop applications can be written in a variety of languages depending on the platform. Mobile applications can be written in Java, Flutter, Swift, etc.

**Server (Application tier)**: the logic tier or middle tier, is the heart of the application. In this tier, information collected in the presentation tier is processed - sometimes against other information in the data tier - using business logic, a specific set of business rules. The application tier can also add, delete or modify data in the data tier.

The application tier is typically developed using Python, Java, Perl, PHP or Ruby, and communicates with the data tier using API calls.

**Database** (**Data-tier**): The data tier, sometimes called database tier, data access tier, or back-end, is where the information processed by the application is stored and managed.

This can be a relational database management system such as PostgreSQL, MySQL, MariaDB, Oracle, DB2, Informix or Microsoft SQL Server, or in a NoSQL Database server such as Cassandra, CouchDB or MongoDB.

## How To Choose The "Suitable" Architecture For Your System?

Well, there are many factors that affect our decisions on how to build systems but we can combine them into two major factors **functional and non-functional requirements**.

### F**unctional requirements and non-functional requirements**

**Functional requirements**

**This is the requirement that the system has to deliver**. We may say it is the main goal of the system. Here a function is described as a specification of behavior between outputs and inputs. What would be system input and what is the output it should be cleared in these requirements. It includes system components, modules, functions.

**Non-functional requirements**

This is the requirement that defines **how we deliver the functional requirements.** These requirements restrict system design through different system qualities.

Performance, modifiability, availability, scalability, reliability, etc. are important quality requirements in system design. These are what we need to analyze for a system and determine if our system is designed properly.

Non-functional requirements play a slightly bigger role than functional requirements.

**Okay, I know what are the functional and non-functional requirements how can they affect system decisions?**

Imagine you are building 2 applications let's stick with the real-time chat app and e-commerce app example.

**Real-time Chat app main non-functional requirements:**

- **Availability**: a chat app is expected to be highly available, You don't want your users to lose access to their messages for a long time if servers went down. **Slack** suffered from the chat service's monumental outage on 4 January and Slack became completely unavailable for its users. [[1]](https://www.theregister.com/2021/02/02/slack_outage_aws_autoscaling/#:~:text=Slack%20says%20it%20has%20identified,reviewing%20the%20TGW%20scaling%20algorithms%22.)
- **Latency**: latency is how much time it takes for a message to travel to its destination. Low latency is crucial in real-time chat apps. You want the users to get their messages in real-time. Reducing the time required to send/receive messages between users is a huge system design factor in this situation.

**E-commerce app main non-functional requirements:**

- **Portability:** it means the ability of a software system to work in a different environment. All e-commerce apps have different clients working in different environments like IOS, Android, Web which allows a variety of users to use the system from anywhere.
- **Security**: Usually all systems require some level of security, e-commerce system often requires more security levels to secure user's data including shipping addresses, payment info (credit card, bank info), etc...

**Most systems require the same non-functional attributes but with different levels, It is all about the trade-offs.**

<table class="table table-striped table-bordered">
<thead>
<tr>
<th>System</th>
<th style="text-align:center; padding:20px">Availability</th>
<th style="text-align:center; padding:20px">Low-Latency</th>
<th style="text-align:center; padding:20px">Portability</th>
<th style="text-align:center; padding:20px">Security</th>
<th style="text-align:center; padding:20px">Scalability</th>
<th style="text-align:center; padding:20px">Compatibility</th>
</tr>
</thead>
<tbody>
<tr>
<td>Real-time Chat app</td>
<td style="text-align:center">Critical</td>
<td style="text-align:center">Critical</td>
<td style="text-align:center">Medium</td>
<td style="text-align:center">High</td>
<td style="text-align:center">Critical</td>
<td style="text-align:center">Low</td>
</tr>
<tr>
<td>E-commerce app</td>
<td style="text-align:center">High</td>
<td style="text-align:center">High</td>
<td style="text-align:center">Critical</td>
<td style="text-align:center">Critical</td>
<td style="text-align:center">Critical</td>
<td style="text-align:center">Meduim</td>
</tr>
</tbody>
</table>



## What Is Scalability And Why It's Important?

Scalability means the ability of the application to handle & withstand increased workload without sacrificing the latency.

For instance, if your app takes x seconds to respond to a user request. It should take the same x seconds to respond to each of the million concurrent user requests on your app.

The backend infrastructure of the app should not crumble under a load of a million concurrent requests. It should scale well when subjected to a heavy traffic load & should maintain the latency of the system.

### **Vertical scaling vs Horizontal scaling**

**Vertical Scaling (scale-up):** It is just the process of adding more resources **(RAM, CPU, Disk storage, etc.)** to your existing server.

let's say if your server is running on 16 GB RAM and suffers from the increased load you increase the RAM to be 32 GB.

**Advantages**:

- It is EASY: You just add resources to your server.
- You don't have to do any code configurations.
- It is a perfect solution for low-traffic servers.

**Disadvantages**

- The serious disadvantage is **hardware limitation:** You will eventually reach the point when you can't add more resources to the server any more It is impossible to add unlimited CPU and memory to a
  single server.
- A single point of failure: **If the server is down that's it, the app goes down.**

![/images/sys-design-basics/vertical-scaling.png](/images/sys-design-basics/vertical-scaling.png)

**Horizontal Scaling (scaling out):** The process of adding more servers to the servers pool.

You make server clones using the same codebase and expose them to users dividing the traffic between the servers.

let's say if your server is running on 16 GB RAM and suffers from the increased load you buy another 16 GB server and share traffic between the two servers.

**Advantages**:

- **No limit to how much we can scale horizontally**: you can buy as many servers as you want.
- **Dynamically scale in real-time as the traffic** on our website increases & decreases over a period of time.
- **There is not a single point of failure:** if one server is down the other servers can handle the requests allowing the app to still up even after one or more servers goes down.

**Disadvantages**

- **Increased Cost**: scaling horizontally will cost you more than vertical scaling.
- **Code Redundancy:** whenever you add a new server you have to install the dependencies and code base of the app.
- **The need for load-balancers add complexity and increases cost:** when you add more servers you will need a way to distribute the incoming requests,

  you don't want one server to get all the requests while other servers are not doing anything. that what load-balancers do

  they evenly distribute incoming traffic among web servers that are defined.

![/images/sys-design-basics/horizontal-scaling.png](/images/sys-design-basics/horizontal-scaling.png)

<img src="/images/sys-design-basics/load_balancer.png" alt="load_balancer" width="700"/>


**In the next post I will discuss load-balancers and how they work**
