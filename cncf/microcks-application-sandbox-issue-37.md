author:	yada
association:	none
edited:	true
status:	none
--
The application for Microcks is ready for review🤞
As stated in the submission above, we have done two presentations to TAG App Delivery:

**Project presentations**

1st presentation to CNCF TAG App Delivery during [pre-day meetup at KubeCon](https://tag-app-delivery.cncf.io/blog/tag-app-delivery-at-kubecon-eu-2023/#pre-day-meetup---tuesday) + CloudNativeCon Europe 2023, see:

- Slide deck ([PDF](https://drive.google.com/file/d/1YY5tIwgcVEjxaYSRfcrsYskaHZaMrHx9/view?usp=sharing) / [Google slide](https://docs.google.com/presentation/d/1DaXkCbXIfSIulLGQMnuuVJpCqcnkC-LLiZfoHcTpe4U/edit?usp=sharing))
- Recording is available here: https://youtu.be/4xzklryIT7Q?t=10291

2nd presentation (2023-05-17) as a follow-up during TAG App Delivery bi-weekly meeting, see:

- CNCF TAG App Delivery [Meeting Notes](https://docs.google.com/document/d/1OykvqvhSG4AxEdmDMXilrupsX2n1qCSJUWwTc3I7AOs/edit#heading=h.2npmztrgu5sw)
- Slide deck ([PDF](https://drive.google.com/file/d/1bBMquCAyY59xsSOvBqzD8nFPlkipMGvL/view?usp=sharing) / [Google slide](https://docs.google.com/presentation/d/1wcX6JHDeaOlJMiwX2PfzgM6JkoU-QY0zPJN6CWpiI0Y/edit?usp=sharing))
- The recording is available here (with a live demo): https://youtu.be/UnNozJiNcz8?t=1599

--
author:	dwojciec
association:	none
edited:	false
status:	none
--
I am writing to express my strong support for the Microcks project, which I believe would be an excellent addition to the CNCF. Microcks is an open-source tool that provides automated API testing, contract validation, and documentation generation for REST and SOAP-based APIs. It is designed to integrate with existing CI/CD pipelines and DevOps toolchains, making it easy to incorporate API testing into your development workflow.

Microcks supports a wide range of API testing frameworks, including OpenAPI, AsyncAPI, and GraphQL, and can generate API documentation in a variety of formats, including Swagger UI, ReDoc, and AsciiDoc. These features make Microcks an incredibly useful tool for developers and organizations looking to improve the quality and reliability of their APIs, and to streamline their API testing and documentation processes.

What's more, I know firsthand that Microcks is highly regarded by its users. Many customers initially considered it a "nice-to-have" tool, but soon realized that it was a "must-have" once they started using it. The feedback I have received from developers and organizations that have implemented Microcks has been overwhelmingly positive, and I believe that it has the potential to make a real difference in the industry.

I strongly encourage the CNCF to consider Microcks for acceptance into the organization. I believe that this project aligns perfectly with the mission and values of the CNCF, and that it could benefit greatly from the resources and support that the organization can offer. Thank you for your time and consideration.
--
author:	yada
association:	none
edited:	false
status:	none
--
Thanks @joshgav for mentioning Microcks in https://github.com/cncf/sandbox/issues/30 comments.
We have noticed the AIP project and will join the next TAG App Delivery meeting on Wednesday June 21 to follow the presentation.
--
author:	joshgav
association:	collaborator
edited:	false
status:	none
--
@yada presented Microcks to TAG App Delivery last month, all the details are in this issue: https://github.com/cncf/tag-app-delivery/issues/411
--
author:	yada
association:	none
edited:	false
status:	none
--
BTW, We have created the #microcks channel in the CNCF slack if you have any questions or would like to know more on the project 🙌 Thanks @amye for the support 👍
--
author:	yada
association:	none
edited:	false
status:	none
--
@amye or @TheFoxAtWork can you add the "App Delivery" label to this submission?
--
author:	TamimiGitHub
association:	none
edited:	false
status:	none
--
Looking forward for this! I too support Microcks being part of the CNCF. Representing the event driven architecture (EDA) industry, I do see great opportunities and potentials of Microcks from asynchronous APIs mocking and testing. There is very limited tooling in EDA for API testing and mocking and Microcks is the most mature in this space. 
--
author:	joshgav
association:	collaborator
edited:	false
status:	none
--
Hi folks - TAG App Delivery worked with @yada at Kubecon EU and at our May 17 meeting to learn all about Microcks; I'll share a report and recommendation here. In short, **if CNCF wants to accept projects focused on cloud-native application development then Microcks is a good fit**.

_Background_: Microcks' capabilities are valuable for developing microservices-oriented, cloud-native applications that communicate synchronously or asynchronously via structured transports and models like HTTP/JSON, GRPC/Protobuf, Kafka/JSON, et al. It aids in constructing such services by letting developers define an interface and semantics without writing a complete implementation of an API service, helpful for API-first development and test simulations in particular.

Microservices applications generally run on cloud-native platforms like Kubernetes or cloud services like AWS ECS or Fargate, and that would be the reason to include projects that support them like Microcks in CNCF. In terms of the functionality it enables, Microcks fits alongside existing projects like Dapr, GRPC and Konveyor in its goals of enabling cloud-native application development. As I mentioned in an earlier comment, its goal of enabling quality microservice/API development is similar to AIP's too (see cncf/toc#30) - perhaps AIP could develop some test cases for their patterns to include in Microcks.

FWIW Microcks might also be compared to CNCF's existing testing frameworks like LitmusChaos and ChaosBlade, though those are focused on resilience and "chaos" testing. Arguably such tests are part of application integration and delivery.

Also relevant, Microcks integrates existing and emerging open standards for API and service development such as OpenAPI, AsyncAPI, GraphQL, CloudEvents, Apache Kafka and more.

My only doubt has been around how CNCF wants to support projects aimed at application developers and cloud-native development. Of course we want them to grow and they're vital to the cloud-native ecosystem, but do we want to host them in CNCF or would they be better off in say Eclipse or Apache?

If CNCF does want to host projects that facilitate cloud-native app development then Microcks would be a great fit. Thank you @yada!
--
author:	yada
association:	none
edited:	false
status:	none
--
Thanks, @joshgav and the TAG App Delivery for the report and recommendation.

To TOC members and @justincormack 👇

Regarding the [TOC review](https://youtu.be/C4bU6A3RToc?t=2777) on June 13, 2023, please see below our answers and point of view regarding the questions/objections raised  :

1/ We do not understand the analogy with the [AIP](https://github.com/cncf/sandbox/issues/30) project, except both are dealing with API!
Microcks is a solution mature and in production which help enterprises to better use the CNCF ecosystem and create cloud-native applications for years see our adopter's usages:
https://github.com/microcks/.github/blob/main/ADOPTERS.md
and the comment from Tamimi: https://github.com/cncf/sandbox/issues/37#issuecomment-1589724813

**IMHO, this is certainly the most important point and we hope that based on the information above and all details in our submission, the TOC can consider adding cloud-native application development tooling as a good fit in the CNCF. Community feedback clearly claims that Microcks helping them use to bring, create and maintain cloud-native workloads at scale is very valuable, see the testimonial from [J.B Hunt](https://www.jbhunt.com/):**
“
Accelerating development
The developers of the project mentioned above saved at least 7 months using Microcks. They were not only able to work concurrently but also captured the exact business requirements specified by the product owner in the form of example request/response pairs. These persistent mocks can now be utilized in sandbox environments if needed.
“
See for the full blog post:
https://microcks.io/blog/jb-hunt-mock-it-till-you-make-it/

2/ Microcks is not a company, we just personally own and use the domain name for the maintainer's emails and project website. We are 100% community-driven and open source.


3/ Microcks is not a Red Hat project or has not been started as a Red Hat project. Microcks has been created in early 2015 and maintainers joined Red Hat in 2017 (both are now Postman employees and joined the [Open Technologies](https://learning.postman.com/open-technologies/) program, see: 
https://microcks.io/blog/microcks-partners-with-postman/

4/ We have already presented twice to TAG App Delivery, comment updated to make it clear here: https://github.com/cncf/sandbox/issues/37#issuecomment-1523746787
BTW, this is a question in the submission form, see:
**Project presentations**
1st presentation to CNCF TAG App Delivery during [pre-day meetup at KubeCon](https://tag-app-delivery.cncf.io/blog/tag-app-delivery-at-kubecon-eu-2023/#pre-day-meetup---tuesday) + CloudNativeCon Europe 2023, see:
- Slide deck ([PDF](https://drive.google.com/file/d/1YY5tIwgcVEjxaYSRfcrsYskaHZaMrHx9/view?usp=sharing) / [Google slide](https://docs.google.com/presentation/d/1DaXkCbXIfSIulLGQMnuuVJpCqcnkC-LLiZfoHcTpe4U/edit?usp=sharing))
- The recording is available here: https://youtu.be/4xzklryIT7Q?t=10291

2nd presentation (2023-05-17) as a follow-up during TAG App Delivery bi-weekly meeting, see:
CNCF TAG App Delivery [Meeting Notes](https://docs.google.com/document/d/1OykvqvhSG4AxEdmDMXilrupsX2n1qCSJUWwTc3I7AOs/edit#heading=h.2npmztrgu5sw)
- Slide deck ([PDF](https://drive.google.com/file/d/1bBMquCAyY59xsSOvBqzD8nFPlkipMGvL/view?usp=sharing) / [Google slide](https://docs.google.com/presentation/d/1wcX6JHDeaOlJMiwX2PfzgM6JkoU-QY0zPJN6CWpiI0Y/edit?usp=sharing))
- The recording is available here (with a live demo): https://youtu.be/UnNozJiNcz8?t=1599

Looking forward to the next steps and let us know if you have any questions or need further information.


Thanks & Regard,
--
author:	amye
association:	none
edited:	false
status:	none
--
/vote-sandbox
--
author:	git-vote
association:	none
edited:	false
status:	none
--
## Vote created

**@amye** has called for a vote on `[Sandbox] Microcks` (#37).



The members of the following teams have binding votes:

| Team |
| ---- |
| @cncf/cncf-toc |





Non-binding votes are also appreciated as a sign of support!

## How to vote

You can cast your vote by reacting to `this` comment. The following reactions are supported:

| In favor | Against | Abstain |
| :------: | :-----: | :-----: |
|    👍     |    👎    |    👀    |

*Please note that voting for multiple options is not allowed and those votes won't be counted.*

The vote will be open for `7days`. It will pass if at least `66%` of the users with binding votes vote `In favor 👍`. Once it's closed, results will be published here as a new comment.
--
author:	abangser
association:	none
edited:	false
status:	none
--
I am writing to express my support in Microcks as a CNCF Sandbox project which seems to fit the criteria of “Independent projects that fit the CNCF mission and provide the potential for a novel approach to existing functional areas (or are an attempt to meet an unfulfilled need).”

I am a part of the the App Delivery TAG and have seen two presentations on this project. The key reasons I see this as a viable Sandbox project is because:

- It builds with the principles held by the TAG (e.g. API driven interactions GitOps enabled deployments)
- Microcks has made a concerted effort to integrate with the larger OSS ecosystem (deployments via operators and helm charts, integrations with Backstage and Keycloak and more).
- Given the CNCF mission to “make cloud computing ubiquitous” and the explicit push for microservices it is important to support and grow projects which enable the success of these architectures by enabling the robust automation to support a loosely coupled architecture.
--
author:	amye
association:	none
edited:	false
status:	none
--
/check-vote
--
author:	git-vote
association:	none
edited:	false
status:	none
--
## Vote status

So far `27.27%` of the users with binding vote are in favor (passing threshold: `66%`).

### Summary

|        In favor        |        Against        |       Abstain        |        Not voted        |
| :--------------------: | :-------------------: | :------------------: | :---------------------: |
| 3 | 0 | 0 | 8 |



### Binding votes (3)

| User | Vote  | Timestamp |
| ---- | :---: | :-------: |
| justincormack | In favor | 2023-06-15 17:15:34.0 +00:00:00 |
| RichiH | In favor | 2023-06-15 7:47:31.0 +00:00:00 |
| TheFoxAtWork | In favor | 2023-06-15 1:23:19.0 +00:00:00 |
<details>
<summary><h3>Non-binding votes (44)</h3></summary>

| User | Vote  | Timestamp |
| ---- | :---: | :-------: |
| angellk | In favor | 2023-06-14 20:33:35.0 +00:00:00 |
| lbroudoux | In favor | 2023-06-14 20:54:38.0 +00:00:00 |
| joshgav | In favor | 2023-06-14 21:00:40.0 +00:00:00 |
| kevinswiber | In favor | 2023-06-14 21:22:13.0 +00:00:00 |
| lanigit | In favor | 2023-06-14 21:23:08.0 +00:00:00 |
| fdavalo | In favor | 2023-06-14 21:54:19.0 +00:00:00 |
| hguerrero | In favor | 2023-06-14 21:57:08.0 +00:00:00 |
| patrickpinto | In favor | 2023-06-14 22:27:39.0 +00:00:00 |
| nmasse-itix | In favor | 2023-06-15 5:25:04.0 +00:00:00 |
| samberthol | In favor | 2023-06-15 5:34:56.0 +00:00:00 |
| lucpierson | In favor | 2023-06-15 5:45:51.0 +00:00:00 |
| nehrman | In favor | 2023-06-15 5:48:22.0 +00:00:00 |
| dwojciec | In favor | 2023-06-15 6:40:39.0 +00:00:00 |
| xymox | In favor | 2023-06-15 7:09:18.0 +00:00:00 |
| meenakshi-dhanani | In favor | 2023-06-15 8:12:27.0 +00:00:00 |
| JulienBreux | In favor | 2023-06-15 9:09:13.0 +00:00:00 |
| arno-di-loreto | In favor | 2023-06-15 9:11:16.0 +00:00:00 |
| Gbahdeyboh | In favor | 2023-06-15 9:16:07.0 +00:00:00 |
| christosgkoros | In favor | 2023-06-15 9:16:34.0 +00:00:00 |
| davidszegedi | In favor | 2023-06-15 9:16:45.0 +00:00:00 |
| arlemi | In favor | 2023-06-15 9:16:51.0 +00:00:00 |
| alexkazan87 | In favor | 2023-06-15 9:18:23.0 +00:00:00 |
| arvindkhadri | In favor | 2023-06-15 9:21:34.0 +00:00:00 |
| Relequestual | In favor | 2023-06-15 9:28:42.0 +00:00:00 |
| hasnain2808 | In favor | 2023-06-15 9:28:59.0 +00:00:00 |
| y-mehta | In favor | 2023-06-15 9:30:49.0 +00:00:00 |
| fmvilas | In favor | 2023-06-15 9:35:31.0 +00:00:00 |
| kaushik-rishi | In favor | 2023-06-15 9:38:13.0 +00:00:00 |
| Souvikns | In favor | 2023-06-15 10:02:48.0 +00:00:00 |
| thulieblack | In favor | 2023-06-15 10:03:49.0 +00:00:00 |
| Shurtu-gal | In favor | 2023-06-15 10:19:02.0 +00:00:00 |
| Barbanio | In favor | 2023-06-15 10:46:38.0 +00:00:00 |
| jonaslagoni | In favor | 2023-06-15 10:51:10.0 +00:00:00 |
| oviecodes | In favor | 2023-06-15 11:08:52.0 +00:00:00 |
| sambhavgupta0705 | In favor | 2023-06-15 11:28:46.0 +00:00:00 |
| Savio629 | In favor | 2023-06-15 11:49:53.0 +00:00:00 |
| carrrson | In favor | 2023-06-15 12:03:15.0 +00:00:00 |
| JBL2011 | In favor | 2023-06-15 12:05:27.0 +00:00:00 |
| AceTheCreator | In favor | 2023-06-15 12:11:03.0 +00:00:00 |
| lserot | In favor | 2023-06-15 12:55:34.0 +00:00:00 |
| salomekbgPostman | In favor | 2023-06-15 14:00:54.0 +00:00:00 |
| EricSaDev | In favor | 2023-06-15 15:40:15.0 +00:00:00 |
| codingmickey | In favor | 2023-06-15 16:04:49.0 +00:00:00 |
| yada | In favor | 2023-06-15 19:23:38.0 +00:00:00 |
</details>



--
author:	hguerrero
association:	none
edited:	false
status:	none
--
Hi everyone! Here is Hugo from Red Hat. I am writing to support the Microcks submission as a sandbox project. 

At Red Hat, we noticed a gap in cloud native solutions for API mocking and testing. That's why we decided to work with Microcks on those areas to enhance our story on API management. I have been contributing for several months to the project, mainly with the Docker Desktop Extension, to the point of becoming the primary maintainer for that component. 

In addition, I have seen the value provided to other organizations using the solution to improve their developer experience. You can find adopters for the project from banks, transport, or the public sector. For this reason, I support their submission.
--
author:	yada
association:	none
edited:	false
status:	none
--
/check-vote
--
author:	git-vote
association:	none
edited:	false
status:	none
--
## Vote status

So far `63.64%` of the users with binding vote are in favor (passing threshold: `66%`).

### Summary

|        In favor        |        Against        |       Abstain        |        Not voted        |
| :--------------------: | :-------------------: | :------------------: | :---------------------: |
| 7 | 0 | 0 | 4 |



### Binding votes (7)

| User | Vote  | Timestamp |
| ---- | :---: | :-------: |
| mattfarina | In favor | 2023-06-16 17:53:51.0 +00:00:00 |
| erinaboyd | In favor | 2023-06-15 20:48:35.0 +00:00:00 |
| RichiH | In favor | 2023-06-15 7:47:31.0 +00:00:00 |
| justincormack | In favor | 2023-06-15 17:15:34.0 +00:00:00 |
| TheFoxAtWork | In favor | 2023-06-15 1:23:19.0 +00:00:00 |
| nikhita | In favor | 2023-06-16 5:42:17.0 +00:00:00 |
| rochaporto | In favor | 2023-06-16 8:23:47.0 +00:00:00 |
<details>
<summary><h3>Non-binding votes (62)</h3></summary>

| User | Vote  | Timestamp |
| ---- | :---: | :-------: |
| angellk | In favor | 2023-06-14 20:33:35.0 +00:00:00 |
| joshgav | In favor | 2023-06-14 21:00:40.0 +00:00:00 |
| kevinswiber | In favor | 2023-06-14 21:22:13.0 +00:00:00 |
| lanigit | In favor | 2023-06-14 21:23:08.0 +00:00:00 |
| fdavalo | In favor | 2023-06-14 21:54:19.0 +00:00:00 |
| hguerrero | In favor | 2023-06-14 21:57:08.0 +00:00:00 |
| patrickpinto | In favor | 2023-06-14 22:27:39.0 +00:00:00 |
| nmasse-itix | In favor | 2023-06-15 5:25:04.0 +00:00:00 |
| samberthol | In favor | 2023-06-15 5:34:56.0 +00:00:00 |
| lucpierson | In favor | 2023-06-15 5:45:51.0 +00:00:00 |
| nehrman | In favor | 2023-06-15 5:48:22.0 +00:00:00 |
| dwojciec | In favor | 2023-06-15 6:40:39.0 +00:00:00 |
| xymox | In favor | 2023-06-15 7:09:18.0 +00:00:00 |
| meenakshi-dhanani | In favor | 2023-06-15 8:12:27.0 +00:00:00 |
| JulienBreux | In favor | 2023-06-15 9:09:13.0 +00:00:00 |
| arno-di-loreto | In favor | 2023-06-15 9:11:16.0 +00:00:00 |
| Gbahdeyboh | In favor | 2023-06-15 9:16:07.0 +00:00:00 |
| christosgkoros | In favor | 2023-06-15 9:16:34.0 +00:00:00 |
| davidszegedi | In favor | 2023-06-15 9:16:45.0 +00:00:00 |
| arlemi | In favor | 2023-06-15 9:16:51.0 +00:00:00 |
| alexkazan87 | In favor | 2023-06-15 9:18:23.0 +00:00:00 |
| arvindkhadri | In favor | 2023-06-15 9:21:34.0 +00:00:00 |
| Relequestual | In favor | 2023-06-15 9:28:42.0 +00:00:00 |
| hasnain2808 | In favor | 2023-06-15 9:28:59.0 +00:00:00 |
| y-mehta | In favor | 2023-06-15 9:30:49.0 +00:00:00 |
| fmvilas | In favor | 2023-06-15 9:35:31.0 +00:00:00 |
| kaushik-rishi | In favor | 2023-06-15 9:38:13.0 +00:00:00 |
| Souvikns | In favor | 2023-06-15 10:02:48.0 +00:00:00 |
| thulieblack | In favor | 2023-06-15 10:03:49.0 +00:00:00 |
| Shurtu-gal | In favor | 2023-06-15 10:19:02.0 +00:00:00 |
| Barbanio | In favor | 2023-06-15 10:46:38.0 +00:00:00 |
| jonaslagoni | In favor | 2023-06-15 10:51:10.0 +00:00:00 |
| oviecodes | In favor | 2023-06-15 11:08:52.0 +00:00:00 |
| sambhavgupta0705 | In favor | 2023-06-15 11:28:46.0 +00:00:00 |
| Savio629 | In favor | 2023-06-15 11:49:53.0 +00:00:00 |
| carrrson | In favor | 2023-06-15 12:03:15.0 +00:00:00 |
| JBL2011 | In favor | 2023-06-15 12:05:27.0 +00:00:00 |
| AceTheCreator | In favor | 2023-06-15 12:11:03.0 +00:00:00 |
| lserot | In favor | 2023-06-15 12:55:34.0 +00:00:00 |
| salomekbgPostman | In favor | 2023-06-15 14:00:54.0 +00:00:00 |
| EricSaDev | In favor | 2023-06-15 15:40:15.0 +00:00:00 |
| codingmickey | In favor | 2023-06-15 16:04:49.0 +00:00:00 |
| yada | In favor | 2023-06-15 19:23:38.0 +00:00:00 |
| luigidemasi | In favor | 2023-06-15 22:34:04.0 +00:00:00 |
| mw1baker | In favor | 2023-06-15 22:34:07.0 +00:00:00 |
| tgubeli | In favor | 2023-06-15 22:40:36.0 +00:00:00 |
| viecili | In favor | 2023-06-15 23:12:08.0 +00:00:00 |
| taku-s | In favor | 2023-06-15 23:47:54.0 +00:00:00 |
| doc-jones | In favor | 2023-06-16 1:25:24.0 +00:00:00 |
| ammachado | In favor | 2023-06-16 3:43:28.0 +00:00:00 |
| Chiledog | In favor | 2023-06-16 3:50:10.0 +00:00:00 |
| rgalanakis | In favor | 2023-06-16 6:25:08.0 +00:00:00 |
| jeanNyil | In favor | 2023-06-16 7:26:39.0 +00:00:00 |
| brunoNetId | In favor | 2023-06-16 7:26:49.0 +00:00:00 |
| tmielke | In favor | 2023-06-16 7:27:02.0 +00:00:00 |
| Croway | In favor | 2023-06-16 7:30:27.0 +00:00:00 |
| sabalde | In favor | 2023-06-16 15:02:28.0 +00:00:00 |
| fmenesesg | In favor | 2023-06-16 15:28:43.0 +00:00:00 |
| CrisCuba | In favor | 2023-06-16 15:30:53.0 +00:00:00 |
| juanliviero | In favor | 2023-06-16 15:31:52.0 +00:00:00 |
| smoya | In favor | 2023-06-16 16:06:40.0 +00:00:00 |
| lbroudoux | In favor | 2023-06-16 19:22:09.0 +00:00:00 |
</details>



--
author:	git-vote
association:	none
edited:	false
status:	none
--
Only repository collaborators can create a vote @ludovic-pourrat.

For organization-owned repositories, the list of collaborators includes outside collaborators, organization members that are direct collaborators, organization members with access through team memberships, organization members with access through default organization permissions, and organization owners.
--
author:	yada
association:	none
edited:	false
status:	none
--
**Regarding Microcks as a good fit in the CNCF?**

📢 We are delighted to have the support of friends and great companies like [Bitergia](https://www.linkedin.com/company/bitergia/) to support our Sandbox CNCF application. Kudos to  [Diane Mueller](https://www.linkedin.com/in/ACoAAAIuxkkBGITIkpaFHjMS2N6vLPRb5njKmV4) for a deep GitHub data analysis regarding Microcks and CNCF connectedness for the last five years! 🚀

See this great visual, that report 10 CNCF projects with contributions by Microcks 🙌

This analysis also reports (for the same period) some very cool metrics from our community :
- 75 Unique Organizations with > 1 contribution
- 151 Participants without Affiliations (Individual contributors)

<img width="1512" alt="CNCF contribs by Microcks" src="https://github.com/cncf/sandbox/assets/552526/72a55cee-a018-4dd1-b06d-40ac73cefc49">

👉 https://www.linkedin.com/feed/update/urn:li:activity:7076562037866147840
--
author:	scottrigby
association:	none
edited:	false
status:	none
--
Hi, I'm part of TAG App Delivery, Adding my support. Hope it's not too late for feedback to be considered.  I was impressed by Yacine's presentation at the meeting. They've put a lot of work into making this a sandbox-worthy project and seem to have the momentum to continue. I'm also excited to play with this in the coming weeks ✊
--
author:	dmueller2001
association:	none
edited:	false
status:	none
--
+1 non-binding  well run community, well connected to upstream projects (see: https://github.com/cncf/sandbox/issues/37#issuecomment-1597284946 for more details); nice plugin built in collaboration with the BackStage community and lots of enterprise traction (aka deployments) giving feedback and testing. Looking forward to seeing where this momentum take them next!
--
author:	amye
association:	none
edited:	false
status:	none
--
/check-vote

--
author:	git-vote
association:	none
edited:	false
status:	none
--
## Vote status

So far `72.73%` of the users with binding vote are in favor (passing threshold: `66%`).

### Summary

|        In favor        |        Against        |       Abstain        |        Not voted        |
| :--------------------: | :-------------------: | :------------------: | :---------------------: |
| 8 | 0 | 0 | 3 |



### Binding votes (8)

| User | Vote  | Timestamp |
| ---- | :---: | :-------: |
| mattfarina | In favor | 2023-06-16 17:53:51.0 +00:00:00 |
| TheFoxAtWork | In favor | 2023-06-15 1:23:19.0 +00:00:00 |
| rochaporto | In favor | 2023-06-16 8:23:47.0 +00:00:00 |
| dzolotusky | In favor | 2023-06-20 2:01:27.0 +00:00:00 |
| erinaboyd | In favor | 2023-06-20 13:09:01.0 +00:00:00 |
| nikhita | In favor | 2023-06-16 5:42:17.0 +00:00:00 |
| justincormack | In favor | 2023-06-15 17:15:34.0 +00:00:00 |
| RichiH | In favor | 2023-06-15 7:47:31.0 +00:00:00 |
<details>
<summary><h3>Non-binding votes (70)</h3></summary>

| User | Vote  | Timestamp |
| ---- | :---: | :-------: |
| angellk | In favor | 2023-06-14 20:33:35.0 +00:00:00 |
| joshgav | In favor | 2023-06-14 21:00:40.0 +00:00:00 |
| kevinswiber | In favor | 2023-06-14 21:22:13.0 +00:00:00 |
| lanigit | In favor | 2023-06-14 21:23:08.0 +00:00:00 |
| fdavalo | In favor | 2023-06-14 21:54:19.0 +00:00:00 |
| hguerrero | In favor | 2023-06-14 21:57:08.0 +00:00:00 |
| patrickpinto | In favor | 2023-06-14 22:27:39.0 +00:00:00 |
| nmasse-itix | In favor | 2023-06-15 5:25:04.0 +00:00:00 |
| samberthol | In favor | 2023-06-15 5:34:56.0 +00:00:00 |
| lucpierson | In favor | 2023-06-15 5:45:51.0 +00:00:00 |
| nehrman | In favor | 2023-06-15 5:48:22.0 +00:00:00 |
| dwojciec | In favor | 2023-06-15 6:40:39.0 +00:00:00 |
| xymox | In favor | 2023-06-15 7:09:18.0 +00:00:00 |
| meenakshi-dhanani | In favor | 2023-06-15 8:12:27.0 +00:00:00 |
| JulienBreux | In favor | 2023-06-15 9:09:13.0 +00:00:00 |
| arno-di-loreto | In favor | 2023-06-15 9:11:16.0 +00:00:00 |
| Gbahdeyboh | In favor | 2023-06-15 9:16:07.0 +00:00:00 |
| christosgkoros | In favor | 2023-06-15 9:16:34.0 +00:00:00 |
| davidszegedi | In favor | 2023-06-15 9:16:45.0 +00:00:00 |
| arlemi | In favor | 2023-06-15 9:16:51.0 +00:00:00 |
| alexkazan87 | In favor | 2023-06-15 9:18:23.0 +00:00:00 |
| arvindkhadri | In favor | 2023-06-15 9:21:34.0 +00:00:00 |
| Relequestual | In favor | 2023-06-15 9:28:42.0 +00:00:00 |
| hasnain2808 | In favor | 2023-06-15 9:28:59.0 +00:00:00 |
| y-mehta | In favor | 2023-06-15 9:30:49.0 +00:00:00 |
| fmvilas | In favor | 2023-06-15 9:35:31.0 +00:00:00 |
| kaushik-rishi | In favor | 2023-06-15 9:38:13.0 +00:00:00 |
| Souvikns | In favor | 2023-06-15 10:02:48.0 +00:00:00 |
| thulieblack | In favor | 2023-06-15 10:03:49.0 +00:00:00 |
| Shurtu-gal | In favor | 2023-06-15 10:19:02.0 +00:00:00 |
| Barbanio | In favor | 2023-06-15 10:46:38.0 +00:00:00 |
| jonaslagoni | In favor | 2023-06-15 10:51:10.0 +00:00:00 |
| oviecodes | In favor | 2023-06-15 11:08:52.0 +00:00:00 |
| sambhavgupta0705 | In favor | 2023-06-15 11:28:46.0 +00:00:00 |
| Savio629 | In favor | 2023-06-15 11:49:53.0 +00:00:00 |
| carrrson | In favor | 2023-06-15 12:03:15.0 +00:00:00 |
| JBL2011 | In favor | 2023-06-15 12:05:27.0 +00:00:00 |
| AceTheCreator | In favor | 2023-06-15 12:11:03.0 +00:00:00 |
| lserot | In favor | 2023-06-15 12:55:34.0 +00:00:00 |
| salomekbgPostman | In favor | 2023-06-15 14:00:54.0 +00:00:00 |
| EricSaDev | In favor | 2023-06-15 15:40:15.0 +00:00:00 |
| codingmickey | In favor | 2023-06-15 16:04:49.0 +00:00:00 |
| yada | In favor | 2023-06-15 19:23:38.0 +00:00:00 |
| luigidemasi | In favor | 2023-06-15 22:34:04.0 +00:00:00 |
| mw1baker | In favor | 2023-06-15 22:34:07.0 +00:00:00 |
| tgubeli | In favor | 2023-06-15 22:40:36.0 +00:00:00 |
| viecili | In favor | 2023-06-15 23:12:08.0 +00:00:00 |
| taku-s | In favor | 2023-06-15 23:47:54.0 +00:00:00 |
| doc-jones | In favor | 2023-06-16 1:25:24.0 +00:00:00 |
| ammachado | In favor | 2023-06-16 3:43:28.0 +00:00:00 |
| Chiledog | In favor | 2023-06-16 3:50:10.0 +00:00:00 |
| rgalanakis | In favor | 2023-06-16 6:25:08.0 +00:00:00 |
| jeanNyil | In favor | 2023-06-16 7:26:39.0 +00:00:00 |
| brunoNetId | In favor | 2023-06-16 7:26:49.0 +00:00:00 |
| tmielke | In favor | 2023-06-16 7:27:02.0 +00:00:00 |
| Croway | In favor | 2023-06-16 7:30:27.0 +00:00:00 |
| sabalde | In favor | 2023-06-16 15:02:28.0 +00:00:00 |
| fmenesesg | In favor | 2023-06-16 15:28:43.0 +00:00:00 |
| CrisCuba | In favor | 2023-06-16 15:30:53.0 +00:00:00 |
| juanliviero | In favor | 2023-06-16 15:31:52.0 +00:00:00 |
| smoya | In favor | 2023-06-16 16:06:40.0 +00:00:00 |
| mayorova | In favor | 2023-06-17 18:17:40.0 +00:00:00 |
| pmbret01 | In favor | 2023-06-18 17:10:05.0 +00:00:00 |
| ludovic-pourrat | In favor | 2023-06-19 11:26:13.0 +00:00:00 |
| matthyx | In favor | 2023-06-19 11:35:31.0 +00:00:00 |
| gilvansfilho | In favor | 2023-06-19 13:48:37.0 +00:00:00 |
| derberg | In favor | 2023-06-19 14:55:51.0 +00:00:00 |
| scottrigby | In favor | 2023-06-19 15:19:30.0 +00:00:00 |
| pittar | In favor | 2023-06-19 18:55:56.0 +00:00:00 |
| lbroudoux | In favor | 2023-06-19 19:33:31.0 +00:00:00 |
</details>



--
author:	git-vote
association:	none
edited:	false
status:	none
--
Votes can only be checked once a day.
--
author:	git-vote
association:	none
edited:	false
status:	none
--
## Vote closed

The vote **passed**! 🎉

`81.82%` of the users with binding vote were in favor (passing threshold: `66%`).

### Summary

|        In favor        |        Against        |       Abstain        |        Not voted        |
| :--------------------: | :-------------------: | :------------------: | :---------------------: |
| 9 | 0 | 0 | 2 |



### Binding votes (9)

| User | Vote  | Timestamp |
| ---- | :---: | :-------: |
| @TheFoxAtWork | In favor | 2023-06-15 1:23:19.0 +00:00:00 |
| @nikhita | In favor | 2023-06-16 5:42:17.0 +00:00:00 |
| @rochaporto | In favor | 2023-06-16 8:23:47.0 +00:00:00 |
| @mauilion | In favor | 2023-06-20 16:55:28.0 +00:00:00 |
| @justincormack | In favor | 2023-06-15 17:15:34.0 +00:00:00 |
| @erinaboyd | In favor | 2023-06-20 13:09:01.0 +00:00:00 |
| @mattfarina | In favor | 2023-06-16 17:53:51.0 +00:00:00 |
| @dzolotusky | In favor | 2023-06-20 2:01:27.0 +00:00:00 |
| @RichiH | In favor | 2023-06-15 7:47:31.0 +00:00:00 |
<details>
<summary><h3>Non-binding votes (73)</h3></summary>

| User | Vote  | Timestamp |
| ---- | :---: | :-------: |
| @angellk | In favor | 2023-06-14 20:33:35.0 +00:00:00 |
| @joshgav | In favor | 2023-06-14 21:00:40.0 +00:00:00 |
| @kevinswiber | In favor | 2023-06-14 21:22:13.0 +00:00:00 |
| @lanigit | In favor | 2023-06-14 21:23:08.0 +00:00:00 |
| @fdavalo | In favor | 2023-06-14 21:54:19.0 +00:00:00 |
| @hguerrero | In favor | 2023-06-14 21:57:08.0 +00:00:00 |
| @patrickpinto | In favor | 2023-06-14 22:27:39.0 +00:00:00 |
| @nmasse-itix | In favor | 2023-06-15 5:25:04.0 +00:00:00 |
| @samberthol | In favor | 2023-06-15 5:34:56.0 +00:00:00 |
| @lucpierson | In favor | 2023-06-15 5:45:51.0 +00:00:00 |
| @nehrman | In favor | 2023-06-15 5:48:22.0 +00:00:00 |
| @dwojciec | In favor | 2023-06-15 6:40:39.0 +00:00:00 |
| @xymox | In favor | 2023-06-15 7:09:18.0 +00:00:00 |
| @meenakshi-dhanani | In favor | 2023-06-15 8:12:27.0 +00:00:00 |
| @JulienBreux | In favor | 2023-06-15 9:09:13.0 +00:00:00 |
| @arno-di-loreto | In favor | 2023-06-15 9:11:16.0 +00:00:00 |
| @Gbahdeyboh | In favor | 2023-06-15 9:16:07.0 +00:00:00 |
| @christosgkoros | In favor | 2023-06-15 9:16:34.0 +00:00:00 |
| @davidszegedi | In favor | 2023-06-15 9:16:45.0 +00:00:00 |
| @arlemi | In favor | 2023-06-15 9:16:51.0 +00:00:00 |
| @alexkazan87 | In favor | 2023-06-15 9:18:23.0 +00:00:00 |
| @arvindkhadri | In favor | 2023-06-15 9:21:34.0 +00:00:00 |
| @Relequestual | In favor | 2023-06-15 9:28:42.0 +00:00:00 |
| @hasnain2808 | In favor | 2023-06-15 9:28:59.0 +00:00:00 |
| @y-mehta | In favor | 2023-06-15 9:30:49.0 +00:00:00 |
| @fmvilas | In favor | 2023-06-15 9:35:31.0 +00:00:00 |
| @kaushik-rishi | In favor | 2023-06-15 9:38:13.0 +00:00:00 |
| @Souvikns | In favor | 2023-06-15 10:02:48.0 +00:00:00 |
| @thulieblack | In favor | 2023-06-15 10:03:49.0 +00:00:00 |
| @Shurtu-gal | In favor | 2023-06-15 10:19:02.0 +00:00:00 |
| @Barbanio | In favor | 2023-06-15 10:46:38.0 +00:00:00 |
| @jonaslagoni | In favor | 2023-06-15 10:51:10.0 +00:00:00 |
| @oviecodes | In favor | 2023-06-15 11:08:52.0 +00:00:00 |
| @sambhavgupta0705 | In favor | 2023-06-15 11:28:46.0 +00:00:00 |
| @Savio629 | In favor | 2023-06-15 11:49:53.0 +00:00:00 |
| @carrrson | In favor | 2023-06-15 12:03:15.0 +00:00:00 |
| @JBL2011 | In favor | 2023-06-15 12:05:27.0 +00:00:00 |
| @AceTheCreator | In favor | 2023-06-15 12:11:03.0 +00:00:00 |
| @lserot | In favor | 2023-06-15 12:55:34.0 +00:00:00 |
| @salomekbgPostman | In favor | 2023-06-15 14:00:54.0 +00:00:00 |
| @EricSaDev | In favor | 2023-06-15 15:40:15.0 +00:00:00 |
| @codingmickey | In favor | 2023-06-15 16:04:49.0 +00:00:00 |
| @yada | In favor | 2023-06-15 19:23:38.0 +00:00:00 |
| @luigidemasi | In favor | 2023-06-15 22:34:04.0 +00:00:00 |
| @mw1baker | In favor | 2023-06-15 22:34:07.0 +00:00:00 |
| @tgubeli | In favor | 2023-06-15 22:40:36.0 +00:00:00 |
| @viecili | In favor | 2023-06-15 23:12:08.0 +00:00:00 |
| @taku-s | In favor | 2023-06-15 23:47:54.0 +00:00:00 |
| @doc-jones | In favor | 2023-06-16 1:25:24.0 +00:00:00 |
| @ammachado | In favor | 2023-06-16 3:43:28.0 +00:00:00 |
| @Chiledog | In favor | 2023-06-16 3:50:10.0 +00:00:00 |
| @rgalanakis | In favor | 2023-06-16 6:25:08.0 +00:00:00 |
| @jeanNyil | In favor | 2023-06-16 7:26:39.0 +00:00:00 |
| @brunoNetId | In favor | 2023-06-16 7:26:49.0 +00:00:00 |
| @tmielke | In favor | 2023-06-16 7:27:02.0 +00:00:00 |
| @Croway | In favor | 2023-06-16 7:30:27.0 +00:00:00 |
| @sabalde | In favor | 2023-06-16 15:02:28.0 +00:00:00 |
| @fmenesesg | In favor | 2023-06-16 15:28:43.0 +00:00:00 |
| @CrisCuba | In favor | 2023-06-16 15:30:53.0 +00:00:00 |
| @juanliviero | In favor | 2023-06-16 15:31:52.0 +00:00:00 |
| @smoya | In favor | 2023-06-16 16:06:40.0 +00:00:00 |
| @mayorova | In favor | 2023-06-17 18:17:40.0 +00:00:00 |
| @pmbret01 | In favor | 2023-06-18 17:10:05.0 +00:00:00 |
| @ludovic-pourrat | In favor | 2023-06-19 11:26:13.0 +00:00:00 |
| @matthyx | In favor | 2023-06-19 11:35:31.0 +00:00:00 |
| @gilvansfilho | In favor | 2023-06-19 13:48:37.0 +00:00:00 |
| @derberg | In favor | 2023-06-19 14:55:51.0 +00:00:00 |
| @scottrigby | In favor | 2023-06-19 15:19:30.0 +00:00:00 |
| @pittar | In favor | 2023-06-19 18:55:56.0 +00:00:00 |
| @lbroudoux | In favor | 2023-06-19 19:33:31.0 +00:00:00 |
| @Damounet | In favor | 2023-06-21 7:41:24.0 +00:00:00 |
| @niklasbeinghaus | In favor | 2023-06-21 12:15:46.0 +00:00:00 |
| @francois-laplante-io | In favor | 2023-06-21 19:56:37.0 +00:00:00 |
</details>



--
author:	amye
association:	none
edited:	false
status:	none
--
Closing with approved, new onboarding issue: https://github.com/cncf/sandbox/issues/197 
--
