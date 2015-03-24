#Global Routing with Anycasted DNS. Teardown: NSONE
#基于任播路由的全球DNS系统--NSONE详解

###What makes NSONE different?
###是什么使得NSONE与众不同？

NSONE is a modern DNS and traffic management platform. At the end of the day, our job is to get your eyeballs to the right place. That can be as simple as answering a plain old DNS query reliably and quickly on a global basis, and as complex as doing detailed computations on the fly to select the best of many potential datacenters, CDNs, or other endpoints to service a user based on all kinds of real-time telemetry about your infrastructure or the internet. To do all that we’ve architected and built a brand new DNS delivery stack from the ground up, deployed a world-class managed DNS network on 6 continents, and fundamentally rethought a lot of how DNS is traditionally used and managed. NSONE’s also an awesome team of hardened infrastructure pros. We’ve all got deep backgrounds across the infrastructure spectrum: hosting, cloud, on-demand bare metal, colo, transit, CDN, you name it, we’ve built it. That breadth of expertise and perspective turns out to be really useful in answering that simple question: where should we send this traffic? It helps that we have a great time while we’re at it, and have a lot of respect for each other and our customers.

NSONE是一个现代DNS和网络流量管理平台。截止到目前为止，我们的工作就是将您的视线拉回到正确的位置。为了完成像在全球范围内快速可靠地响应查询域名这样简单的任务,或者像在飞行中做详细的计算并从众多的数据中心中选出最优解，CDN系统，或为了使得终端用户能够实时遥测基础设施或互联网这样复杂的任务，我们从头架构和搭建了一个新的DNS栈，并在6各大洲部署了一个世界级的DNS网络，从根本上重新思考了很多关于传统的DNS是如何使用和管理的。NSONE也是一个在巩固基础设施方面很棒的团队。我们在网络基础设施方面有很强的技术背景：网站托管，云服务，按需裸机编程，托管中心，通信，只要您能说出来的，我们都做过。所以，在回答我们该把流量转向哪里这个简单的问题上，我们渊博的专业知识就变得很有用了。它们帮助我们处于并将持续处于良好的状态，而且拥有很多彼此尊敬的客户。

###How do you think about DNS based routing, and what are some of the interesting ways you route traffic for your customers?

###请问您如何看待一个基于路由的DNS以及您为客户做路由转发时有什么有意思的方法？

If you’re just delivering your application from a single datacenter, DNS based routing isn’t something you’re thinking about. But the moment you’re in multiple datacenters and you’ve got a choice — which endpoint is going to best service a user — things get interesting. “Best” is a totally overloaded term: it might be something like fastest response times, or highest throughput, or lowest packet loss, but it could also be any combination of performance and business metrics.

如果您只从一个单个数据中心传递应用数据，那么基于路由的DNS不是您所要考虑的。但有趣的是，当您面对多个数据中心时，必须做出选择：哪个终端是服务一个用户最佳的终端？"最佳"是一个可以重载的词：它可能指最快的响应时间，最高的吞吐量，或者最低的丢包率，也可能是任何性能与业务指标的组合。

Most DNS based routing that’s being done today makes some simplistic assumptions, for example, that the datacenter physically closest to a user is the one that’s going to give the best service. But that’s not how the internet works, and it’s also a very myopic view of “best”.

如今大多数已实现的基于路由的DNS都做了一些简单的假设，比如，物理距离到一个用户最短的数据中心会提供最佳的服务。实际的互联网并非如此，这个假设是一种很短视的“最佳”。

Our view is that at decision making time, when we get a DNS query for some domain and have a bunch of potential answers to select from — usually service endpoints for your application — we should have as much information on hand as we possibly can about those endpoints.

我们的观点是：当从一些终端用户的常用应用中得到DNS查询请求后，可能需要从一大堆的候选答案中选择时，手头需要有尽可能多的信息做参考。

Mostly this information falls into three categories:

大多数情况下这些信息可以被分成3类:

1.Static details: stuff like where your servers are located, how many cores they have, basic priorities and weights, etc.

1.静态细节信息：比如您的客户在哪，他们有多少骨干路由器，原始的优先级和权重等等。

2.Infrastructure metrics: real-time information about what’s happening in your infrastructure, like load averages, connection counts on load balancers, how much of your commits you’ve used, and so on.

2.基础设施上的参数：基础设施上所产生的实时信息，如平均负载，负载均衡器的连接数，使用过的请求数量等等。

3.Eyeball metrics: real-time information about what’s happening between end users and your endpoints, like granular latency, throughput, or similar metrics from the vantage point of end users, or information about the state of the global routing table, etc.

3.可以看到的指标：您的用户和终端节点所产生的实时信息，如颗粒的时延，吞吐量，类似的从终端客户角度看最终的数据，或者全局路由表的状态等等。

What we’ve built is a set of systems and a platform that’s meant to take in all this information, often at high frequency, massage it into something useful for routing, and get it out to our DNS delivery edges as close to real-time as possible so it can be used in making routing decisions.

为了能够在高频度请求的环境下，得到用于近乎实时响应路由决策的所需要的所有信息，我们建立起了一套的系统和平台。

It’s one thing to have a lot of data available to use in routing; it’s another thing to make it easy to use. This is what our Filter Chain tech is all about. At a high level, the Filter Chain is like a sequence of little building block “filter” algorithms. Each filter examines the set of answers we could give to a DNS query, and all the static config and metrics we have, and manipulates the set of answers somehow: remove endpoints that are down, sorts them by response time for the network of the requester, sheds load from overloaded servers, etc. When you connect a bunch of these filters in sequence it turns out to be really easy to build complex turnkey routing setups driven by real-time data about your application infrastructure.

我们使用的过滤链技术完成了两件事:1.获取到海量用于路由的数据；2.高效的利用这些数据。抽象地讲，整个过滤链就像由一系列的“积木过滤算法“组成的大厦一样。每个过滤器检查我们回答一个的DNS查询的答案集合，以及我们所拥有的静态配置和指标，然后以某种方式操作这个集合:删除宕机的节点，按照响应时间进行排序，对超负载的服务器卸负载等等。当您把这些过滤器按序相连后，就会很容易为基础设施应用建立起一个由实时数据驱动的复杂路由安装程序。

Our customers use the tech in all sorts of interesting ways, and are always finding new ways of mixing and matching data and algorithms to get powerful behaviors. It’s common for us to do something like: pick a region (e.g. US-WEST or US-EAST), then within the region, send 95% of traffic to colo and 5% to AWS, and make that sticky so most of the time the same 5% of users to go AWS to ensure good cache locality, and if colo infrastructure gets overloaded, start shifting weight so more traffic goes to AWS with auto-scaling enabled, and on and on.

我们的客户以很多有趣的方式使用这些技术，经常发现新的方法融合和匹配数据进行计算，从而得到更好的效果。我们通常会做类似这样的事情：选择一个区域(比如 美国西部或东部)，在这个区域内，向托管中心转发95%的流量，向AWS转发发5%的流量，保持这个比例不变，保证了5%的使用AWS用户有高效的缓存可用，如果托管中心负荷满了，开始转换比例，使得更多的流量转到AWS，从而开启自动规模化，等等。

These capabilities are naturally suited to hyper performance sensitive applications in many datacenters, so we do a lot of routing for CDNs, ad tech companies, major web properties, SaaS platforms, and the like. But really, the moment you graduate from a single datacenter app deployment into two or more datacenters, we can make a powerful difference.

这些能力很适合部署在许多数据中心中的性能敏感的应用程序，所以我们为CDN系统，广告技术公司，网络性能提供商，SaaS平台等做了很多路由工作。但说真的，当您学会了在将应用程序部署到单一的数据中心，两个或两个以上的数据中心的时候，我们可以做得更多。

###What is your stack?

###您的技术栈是什么？

There’s a lot going on in our stack. On the delivery side, we’re touching stuff from the hardware/NIC level (crazy packet filtering), doing deep traffic engineering in BGP, leveraging low level kernel features to get as precise as routing DNS queries to specific cores to maximize cache locality, and hitting a totally custom written nameserver that executes complex routing algorithms for every single request. At a higher level, what we’ve built is a big globally distributed real-time system, and we’ve tried to use the right tools for the right jobs. You’ll see some  Mongo (it’s good at replication but we are lightweight with it on the reads/writes),  RabbitMQ (which helps us quickly propagate config changes and routing data to every nameserver in our network),  Redis, etc. In our core facilities where our portal, API, and other central systems live, we’re using stuff like  Hbase and OpenTSDB for metrics. We have a lot of our own software, much of it  Python (very heavily Twisted). Probably 20+ different roles. Everything is managed by  Ansible, which is really crucial to the velocity with which we iterate and deploy. And we have a really solid QA/build framework and process in place, with a comprehensive functional testing suite. In something as mission critical as DNS you need to be pedantic about QA.

在数据传输方面，我们接触了从硬件/网卡层(超强的包过滤)，使用BGP进行深度流量工程，利用底层内核特性得到精确的DNS路由查询，到最大化缓存位置，以及编写一个完全自定义的域名服务器来针对每个请求执行复杂的路由算法等技术。在偏向应用层上，我们在全球建立了一个分布式实时系统，我们试着使用合适的工具做事情。你会看到MongoDB(很擅长复制数据，但我们用来做轻量级的读/写)，RabbitMQ(帮助我们快速地将网络中每台域名服务器的配置更改和路由数据进行传递)，Redis等等。在我们的门户网站，API和其他中心系统所在的核心设备上，我们使用像Hbase和OpenTSDB这样的工具，我们使用很多自己的软件，多数是Python语言编写的(严重依赖Twisted框架)。所有玩意都由Ansible管理，它对我们快速迭代和部署来讲很重要。此外,我们有有很健壮的QA框架,提供了很全面的功能测试套件。类似于DNS这样的任务，您需要近乎苛刻的测试来保证质量。

###What tools do you use for performance Monitoring?
###您使用了哪些工具监控性能？

Catchpoint is our primary monitoring tool. We also spend a lot of time with route-views and various backbone looking glasses just to ensure we have a clear view of how traffic is getting to our network. Going forward, I anticipate we’ll be using CloudHelix more and more — it’s a powerful tool for doing ad hoc queries on flow metrics.

Catchpoint是我们基本的监控工具。我们也花大量时间来观察路由和很多骨干网来保证我们对网络走向有一个清楚的认识。以后，我期望我们将越来越多地使用cloudhelix--一款即时查询，流量指标的有力工具。

We generally avoid using some of the simple online ping and other tools: we frequently find they have incorrectly geolocated their nodes, or that they’re otherwise not producing very reliable results. We also have a very significant RUM infrastructure of our own as part of our higher end routing tech, and there are some unique ways we can leverage that to get very granular data on the performance of our DNS infrastructure across a breadth of end user networks.

通常，我们会避免使用一些简单的在线ping等工具，我们经常发现他们的节点定位错误或者提供的结果不可靠。我们在路由技术中使用了自己卓越的路由单元管理设施，并利用一些独特的方式从广泛的网络终端用户那里，得到有关我们的DNS的精准数据。

###How is your network laid out?
###您的网络是如何布局的？

This is a fun question, because NSONE operates a whole variety of networks. One of the truly unique things we do is deploy private, dedicated managed DNS networks powered by our delivery stack, so there are all sorts of topologies at play. Our big, global, managed DNS network — the one most of our customers use — is spread across six continents and 17 POPs (with some more on the way). It’s anycasted, with around 7-8 carriers in the mix depending on the market. We make really heavy use of BGP communities and specifically work with providers that have solid BGP community policies in place, so we can do really precise traffic engineering to tune our anycasting. It took a lot of work, but now we have one of the fastest DNS networks on the planet.

这是个有趣的问题，因为NSONE操作的是一个多样化的网络。我们所部署的技术栈用来给私有专用网络提供DNS网络管理系统，所以使用了很多技术。我们的大多数客户使用的是一个跨越6个大洲和17个拨入点(以后会有更多的)。它是一个任播网络，根据市场不同大约有7到8个运营商。我们很依赖BGP的Community属性值，而且和拥有可靠BGP协议语义的提供者合作，所以我们真的能做到精确流量工程来调整我们的选播。NNSONE要承担很多工作，但是现在我们拥有这个这个星球上最快的DNS网络之一。

When you’re building a global anycasted network, what you really care about is topological distance to carriers — especially, to all the big backbones. When we announce our prefixes to a particular network, we need to tune the announcements very precisely to make sure all our global announcements are equidistant (same number of hops) from the backbones. This is where BGP communities come in: we hint to our upstreams to prepend hops to certain routes, or prevent export of our prefixes to certain NSPs. In some regions like South America or Africa, we need to be really careful to prevent export of our routes outside the region. And we always need to prevent route “leakage” via peering exchanges or other paths that some networks normally prefer.

当您要构建一个全球化的任播网络时，您真正需要关注的是各个运营商之间的拓扑距离，尤其是到骨干网的距离。当我们公告到特定网络的前缀时，需要精确地调整参数确保全球到骨干网是等距的（同样的跳数）。这就是BGP中community值的由来：指定上游流量到确定的路由器，或者组织流量到特定的供应商。在一些地区，比如南美或非洲，我们需要确保那些流量没有走出该区域。此外，我们通常还要避免一些网络喜欢的通过对等交换实现的网络漏洞。

We’re adding locations all the time.

When we deploy private DNS networks, they can be pretty unique. Commonly we’ll deploy into a customer’s existing datacenter footprint and use their connectivity. Sometimes we build purely internal-facing DNS networks for service discovery or other purposes. We also run into quite a few customers who need a real globally distributed, anycasted network, but for regulatory, policy, or technology reasons they can’t use our managed DNS network, and they don’t already have their own global footprints to leverage. We can deploy those kinds of networks into a few clued-in public cloud providers like HostVirtual. We even deploy right into Black Lotus’s DDoS scrubbing centers for customers that have unique DDoS requirements.
We use ExaBGP and some custom code to automate some of the communities. ExaBGP includes support for flowspec which is not supported in a lot of networks, but where it is, is a powerful/flexible way to have really fine-grained routing control.

我们一直在扩展服务区域。

当我们部署私有DNS网络时，它们可能相当独特。通常我们会将网络部署到客户现有的数据中心并使用他们的连接。有时候，我们会构建一个纯粹用于服务扩展或其他目的的内部DNS网络。我们也遇到了不少的客户需要一个真正的全球分布式网络，但是由于监管，任播，政策，或技术的原因他们不能使用我们的DNS网络管理，并且他们还没有自己的全球数据作为支撑。我们可以将这些不同类的网络部署到像HostVirtual这样的共有云平台上。对于有特殊防DDoS攻击的用户，我们还会将网络部署到Black Lotus的防DDos过滤数据中心。

We use ExaBGP and some custom code to automate some of the communities. ExaBGP includes support for flowspec which is not supported in a lot of networks, but where it is, is a powerful/flexible way to have really fine-grained routing control.


我们使用ExaBGP和一些客户的代码来自动化一些community值。ExaBGP带有一些很多网络不支持的特性，但是正是这些特性，提供了灵活强大的控制路由的能力。

##What is the make up of your team?
###您的团队由哪些人组成?

We have an amazing team. A lot of us came out of a successful infrastructure company called Voxel that was bought by Internap a few years ago. We’ve worked together for close to a decade, specifically on high volume internet infrastructure. We’re a small team — a couple engineers, ops/devops/neteng to manage our global network, a bit of sales and marketing, and customer support. Everyone on our team spends time with customers — that’s something we really value.

我们的团队很棒。大多数人从一个在网络基础设施方面很成功的公司Voxel走出来的，那家公司几年前被Internap收购了。我们公事了近十年，特别是高容量的网络基础设施方面。我们是一个小团队，由工程师，OPS / DevOps / neteng来管理我们的全球网络，还有销售，市场营销，和客户支持。每个人都在和客户一起找到我们的价值。

###What is the workload being processed?
###正在处理的工作负载是什么?
On the delivery side, our fundamental unit of work is the DNS query. For every query, we’re executing a custom sequence of routing algorithms (which we call the Filter Chain) acting on a collection of answers (e.g., load balancer IPs or CDN hostnames) and realtime data about the answers (e.g. load or other infrastructure metrics, network metrics, etc). The other less visible workload for us is on the data side: ingesting, classifying, normalizing, aggregating, and distributing volumetric, real-time data that’s useful for routing. Sometimes, thousands of data points per second are flying into our systems, and we need to be really smart about how and when to send the data out to our far-flung edges for use by the Filter Chain.

在交付端，我们的工作的基本单位是DNS查询。对每一个查询，我们在结果集（例如负载均衡器的IP或者CDN的域名）或是结果集的实时数据（如负载或其他基础设施指标，网络指标等）上执行一系列自定义的路由算法（我们称之为过滤链）。我们的其他不可见的工作量在数据方面：提取，分类，标准化，聚集和分布是存储我们用于路由的实时数据。有时候，每秒上千的节点的数据进入我们的系统，我们需要巧妙地决定如何以及何时将数据发给过滤链来使用。

###How important is latency to you?
###时延对您来说有多重要？

Latency is important in a lot of ways. The traditional managed DNS differentiator is basically how fast you spit out an answer. It’s table stakes in our industry to spit out an answer really, really fast. What’s much more interesting to us and where we think the DNS and traffic management industry needs to head is spitting out the *best* answer. More and more, applications are built to be distributed from the ground up, whether for reliability (DR environments) or performance (pushing the application close to the edge). It’s not that hard anymore to spin up a new CDN of your own using a modern cloud provider and nginx. Nor is it that hard to solve the multi-datacenter data replication and consistency problem with a whole raft of modern databases. It’s even fairly easy to use the basic geoip features of most managed DNS providers to do some simple, decent routing to get eyeballs to the “closest” datacenter. But if you’re really trying to optimize your application’s latency, or throughput, or really any business metric, second order approximations like geographic proximity won’t do — that’s not how the internet actually works. You need to measure those metrics: what’s the response time of a user on Verizon in New York to each of your different datacenters, right now? And you need to suck that data in at high frequency, do a bunch of math, and use it to send users to the right place. Which is what we do. So yeah: latency is pretty important to us. But maybe not in the way you’d think, at least at first glance.

从很多角度看，时延很重要。传统的DNS管理的功能就是快速给出应答。很快地给出响应是做这一行的关键。我们正在思考的一个更有趣的问题是，DNS和流量管理应该把给出“最佳”答案放在首位。不仅如此，更多的分布式应用应运而生，不管从可靠性（DR环境）还是性能（极限情况下）。在云服务和Nginx的帮助下，新建一个CDN不再是难事。在多个现代数据中心解决数据一致性问题也不再是难事。使用一些基本的网络供应商的GEOIP特性来做一些简单的，花哨的路由工作来得到一些数据中心的认可也是容易的。但如果您真的想优化您的应用程序的延迟，吞吐量，或任何真正的业务指标，粗略使用地理接近就能给出好的服务来看，并不是网络实际的工作方式。您需要衡量以下指标：现在的纽约用户在Verizon对你的每一个不同的数据中心，响应时间是什么？您需要高速地获取这些数据，做一些计算，并把他们放到正确的位置。这些是我们做的。所以：延迟对我们是很重要的。但也许不是你想的，至少不是一开始那样的。

###What’s the current state of edns-client-subnet on the internet?
###目前，edns-client-subnet情况是怎样的？

For those that aren’t familiar, edns-client-subnet (ECS) is an extension to the DNS protocol. DNS resolvers that support it (like Google Public DNS and OpenDNS) will include additional details about the original requester when sending a query to an authority (like NSONE) that supports ECS — usually, the first three octets of the requester’s IP address are sent along. That information can be really useful in doing traffic management, especially if you’re doing geoip or some of the more advanced routing NSONE does, since you have some information about the actual end user instead of just some big centralized resolver they’re using.NSONE is one of the few DNS providers that supports ECS, and we’ve been doing it for a while now. We’ve learned some interesting stuff.

这些不尽相同，edns-client-subnet (ECS)是DNS协议的一个扩展。DNS解析器，支持edns-client-subnet（像谷歌公共DNS和OpenDNS），包括关于原始请求更多的细节，一个权威的发送一个查询时（如nsone）支持ECS。通常，请求者的IP地址的第一个三字节一起发送。这些信息可以在流量管理是非常有用的，尤其是如果你做GeoIP或者像NSONE一样做更先进的路由，因为你有一些实际的终端用户的信息而不是一些庞大解析器生产的。

NSONE是少数支持支持ECS的DNS提供商，我们已经做了一段时间了。我们学到了一些有趣的东西。

We pretty much only see ECS-enabled queries from Google and OpenDNS. Very few other resolvers support it. But mostly, that’s okay: Google/OpenDNS are big anycasted global resolver networks with nodes that might not really be that related (geographically or otherwise) to actual end users, so it’s valuable to get additional information from them for routing. The majority of other DNS resolvers tend to be more correlated with actual end users so there’d be less benefit in enabling ECS on those.

除了Google和OpenDNS以外，我们很少看到开启ECS的DNS服务。其他解析器很少支持它。但是，这没关系，谷歌/OpenDNS的庞大解析器全球解决网络节点可能并不是真正和用户相关的(地理上或其他方面)，因此，从他们身上得到信息用于路由是很有必要的。大多数的DNS解析器更倾向于服务终端用户，因此ECS对他们没什么大用。

About 10-15% of the DNS queries we get have ECS data attached. What’s interesting is the moment we start returning query responses that use ECS data, the number of ECS-enabled queries increases because resolvers can only cache responses with respect to the subnet (usually /24) the response applies to. If you have a DNS record like an ad pixel domain that’s hit from most of the subnets on the internet, ECS can lead to a lot more DNS traffic (but also much better routing accuracy). If you have a record that’s localized in its set of users, ECS won’t affect your DNS traffic much. In general, we can do much more precise routing for ECS-enabled queries than we’d otherwise be able to do, thanks to the prevalence of Google Public DNS and OpenDNS.

我们收到的DNS查询中有大约10-15%的带有ECS数据。有趣的是，当我们开始返回查询的响应时候，使用ECS数据，使查询的数量增加，因为解析器只能缓存响应与子网（通常是/24）。如果你有一个类似于一个广告域的DNS记录，ECS会导致更多的DNS流量（但是也会有更好的路由精度）。如果你有一个记录，是用于定位其用户组，ECS不会影响流量的增加。一般来说，我们使用ECS来使得DNS查询的路由更加精确，多亏了Google公用DNS和OpenDNS的流行。

###What’s next for NSONE?
###NSONE下一步要做什么？

A lot of top secret stuff! We’re hiring like mad (isn’t everybody?) because we’re growing so fast. We’ve found a really fun space to be working in and we’re trying as hard as we can to really push the boundaries of what’s possible in traffic management. That means things like true eyeball metrics based routing, bespoke software defined private DNS networks specifically tailored to application workloads, and a lot of new and powerful algorithms and tools for doing and thinking about DNS and traffic management.

很多绝密的东西！我们疯了一样招人(难道每个公司不是这样的？)，我们成长的很快。我们发现一个非常有趣的领域，我们正在尽我们所能去推动在流量管理中什么是可能的边界。这意味着像真正的可视化度量的路由，定制的软件，自定义的私有DNS网络以及专门定制的应用程序，许多新的和强大的算法工具以及对DNS和流量管理的思考。
