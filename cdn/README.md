# Summary

**DNS**: tree-based, flexible, fast domain-name to IP-address mapping. URL mapping captures the tree-level - resolution happens recursively, in reverse of the URL. The resolution of each part of the URL can happen recursively (load on namespace server) or iteratively (load on querier).

DNS cache improves performance. Cache validity through TTL field for entries.

**Bailiwick checking** - verify reply of DNS server to make sure domain returned via cache in additional section has the same name as requested.

HTTP 1.1 Improvements: Keep-alive the TCP connection instead of 1 connection per object. With slow-start, this helps faster download. Pipelining object retrieval overlaps RTTs on requests.

**Load balancing**: use DNS round robin and multiple records or have a NAT-like load balancer.

Network traffic still high - reduced with web caching. **Reverse proxies** only reduce server load. Both server ISP, backbone ISP, and client ISP still get load. **Forward proxies** are inside client ISP. Issues: **flash crowds**, low cache hit rates.

CDN: replicate an origin server; updates pushed to replicas.

![akamai dns](/cdn/akamai.png)

Akamai DNS use for setting up CDN nodes.

0. Client issues request to content provider. Content provider links to Akamai-cached versions of its objects.
0. Content provider replies (via its own server or proxy)
0. Client browser resolves links for first time, hitting top-level domain DNS.
0. DNS replies with Akamai global DNS
0. Client recurses on Akamai global DNS
0. Akamai global responds with topographically-chosen regional DNS (i.e., so client has fastest access).
0. Client goes to regional Akamai DNS
0. Regional DNS responds with best guess for local Akamai cluster near client
0. Client finally goes to endpoint in Akamai cluster
0. Cluster serves client the data

From then on, on cache hits, since the regional Akamai DNS will have a long TTL, the client will just go in the following sequence:

0. Raw HTML from provider
0. Straight to regional DNS
0. Served by cluster.

Akamai secret sauce comes from clever IP equivalence classes construction from various metrics to maximize hit rate and reduce traffic. Entries inside the regional server have short TTL for this reason (better metrics).
