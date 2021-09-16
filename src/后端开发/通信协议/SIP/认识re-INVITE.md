
## 定义

RFC3261在第14节指明：

> An INVITE request sent within an existing dialog is known as a re-INVITE.


意思是re-INVITE是对话（Dialog）内发出的INVITE，其本质为INVITE，即re-INVITE不是新的SIP method（方法）。


## 功能

我们知道INVITE是用来初始化一个会话（Session），RFC3261在第13节中的原话如下：

> When a user agent client desires to initiate a session (for example,   audio, video, or a game), it formulates an INVITE request.  The INVITE request asks a server to establish a session.
……
A 2xx response to an INVITE establishes a session, and it also creates a dialog between the UA that issued the INVITE and the UA that generated the 2xx response.


且第14节又简洁地复述了一遍：

> A successful INVITE request establishes both a dialog between two user agents and a session using the offer-answer model.


一个会话建立后还可以修改吗？回答是肯定的，如何修改？这就需要re-INVITE，第14节中的原话如下：

> This modification can involve changing addresses or ports, adding a media stream, deleting a media stream, and so on.  This is accomplished by sending a new INVITE request within the same dialog that established the session.
……
Note that a single re-INVITE can modify the dialog and the parameters of the session at the same time.


简而言之，**INVITE用来建立会话的，而re-INVITE是用来修改会话的**。


## 识别re-INVITE

既然re-INVITE的本质就是INVITE，那对于一个SIP报文，我们怎么辨别它是INVITE，还是re-INVITE呢？

如果您对对话和会话的建立过程了解的话，区分INVITE和re-INVITE其实是非常容易的。

您知道什么叫SIP对话吗？RFC3261在第4节的概述中就对对话下了个定义：

> The combination of the To tag, From tag, and Call-ID completely defines a peer-to-peer SIP relationship between Alice and Bob and is referred to as a dialog.


对！请牢记在心：对话的三要素就是**To tag，From tag和Call-ID**。

显然，用来初始化会话的INVITE发出时，UAC只知道To头，而不知道To tag。也就是说，这时INVITE中有From，From tag，To和Call-ID，就是没有To tag，这很正常，此时对话尚未建立，UAS还没有参与进来，而To tag正是UAS产生的，正如From tag产生于UAC。

这一点，RFC3261在第8.1.1.2小节描述To头明确指出：

> A request outside of a dialog MUST NOT contain a To tag; the tag in the To field of a request identifies the peer of the dialog.  Since no dialog is established, no tag is present.


那有人可能会问：什么时候会出现To tag，是不是UAS刚开始回应时就携带To tag呢？答案是不一定，RFC3261在第8.2.6.2小节指明：

> However, if the To header field in the request did not contain a tag, the URI in the To header field in the response MUST equal the URI in the To header field; additionally, the UAS MUST add a tag to the To header field in the response (with the exception of the 100 (Trying) response, in which a tag MAY be present).  This serves to identify the UAS that is responding, possibly resulting in a component of a dialog ID.  The same tag MUST be used for all responses to that request, both final and provisional (again excepting the 100 (Trying)).


读到这里，您肯定知道，既然re-INVITE是用来修改会话的，**在对话内发送，其中肯定含有To tag**。


## 结论

以上罗嗦了很多，总结起来就是一句话，那就是：

**INVITE报文中肯定没有To tag，而re-INVITE中肯定含有To tag**。

---

