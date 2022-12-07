# MFE - Circling the Square!

## Integration Techniques

Build time integration not recommended (see [here](https://martinfowler.com/articles/micro-frontends.html#Benefits)) - this is our current approach, and probably the most practical approach but we should be aware of the trade-offs.

Run time integration is more complex but gives benefits around looser coupling, ability to publish new MFEs entirely independently to production, ability to use different libraries and versions without fear of breaking other aspects of the application

Runtime downsides: potential library bloat, i.e. duplicate downloads of libraries to clients (although can be managed using something like shared modules via webpack's module federation), unable to offer SSR and SSG pages (at least to the best of my knowledge) which undermines one of the major use cases for selecting NextJs?



## Decoupling

The MFE approach promotes decoupling the full stack of development through vertical slices of the application.

Our current approach uses micro services for the backend (generally integrating with 3rd party vendors for persistence and domain specific logic), these micro-services are consolidated into a singular GraphQL Backend for Front end layer (specifically designed to be consumed by the monolithic web app).  The web-app then queries and mutates data via the BFF.  The diagram below illustrates the current set up

```mermaid
flowchart TB
	WebApp([WebApp]) <--> BFF([BFF])
	BFF([BFF GraphQL Server]) <--> CustDS([Customer Domain Service])
	CustDS <--> Gigya([SAP CDC])
	BFF <--> ContDS([Content Domain Service])
	ContDS <--> CS([Content Stack])
	BFF <--> ProdDS([Product Domain Service])
	ProdDS <--> CT([Commerce Tools])
	BFF <--> SrchDS([Search Domain Service])
	SrchDS <--> AT([Attraqt])
	BFF <--> CartDS([Cart Domain Service])
	CartDS <--> CT
```

The current proposed approach to Micro-Frontends seeks to split the WebApp into Feature Apps and Feature Services, but retains the singular BFF.

Thus we end up with something like this:
```mermaid
flowchart TB
	Container([Container Web App])
	Container -.- FootFA([Footer Feature App])
	Container -.- HeadFA([Header Feature App])
	Container -.- HomeFA([Homepage Feature App])
	Container -.- SearchFA([Search Feature App])
	Container -.- PLPFA([PLP Feature App])
	Container -.- PDPFA([PDP Feature App])
	Container -.- CartFA([Cart Feature App])
	Container -.- CheckoutFA([Checkout Feature App])
	Container -.- CustFA([Customer Feature App])
	FootFA -.- CatFS([Categories Feature Service])
	HeadFA -.- CatFS
	HeadFA -.- SearchFS
	HeadFA -.- CartFS
	HeadFA -.- CustFS
	HomeFA -.- HomeFS([Home Feature Service])
	SearchFA -.- SearchFS([Search Feature Service])
	PLPFA -.- PLPFS([PLP Feature Service])
	PDPFA -.- PDPFS([PDP Feature Service])
	CartFA -.- CartFS([Cart Feature Service])
	CheckoutFA -.- CheckoutFS([Checkout Feature Service])
	CustFA -.- CustFS([Customer Feature Service])
	BFF([BFF GraphQL Server])
	CatFS<-->BFF
	HomeFS<-->BFF
	SearchFS<-->BFF
	PLPFS<-->BFF
	PDPFS<-->BFF
	CartFS<-->BFF
	CheckoutFS<-->BFF
	CustFS<-->BFF
	BFF([BFF GraphQL Server]) <--> CustDS([Customer Domain Service])
	CustDS <--> Gigya([SAP CDC])
	BFF <--> ContDS([Content Domain Service])
	ContDS <--> CS([Content Stack])
	BFF <--> ProdDS([Product Domain Service])
	ProdDS <--> CT([Commerce Tools])
	BFF <--> SrchDS([Search Domain Service])
	SrchDS <--> AT([Attraqt])
	BFF <--> CartDS([Cart Domain Service])
	CartDS <--> CT
```

Unless I've misunderstood the intended implementation, the current proposed approach to MFEs will result in a tangle of dependencies and a singular monolithic BFF Server.

A cleaner approach would be to avoid the sharing of Feature Services across different Feature Apps, each Feature App should have it's own Feature Service.  This will be some duplication of code, however, it reduces coupling (one of MFE's key benefits).  It's likely that one shared Feature Service will remain to handle such cross cutting concerns as customer and authentication.  In this version the singular BFF is retained, as it may have some benefits around state management and caching - **TBC**

Feature Services are simply used to abstract business logic and data access concerns out of React components - which should be centred on presentation concerns.



```mermaid
flowchart TB
	Container([Container Web App])
	Container -.- FootFA([Footer Feature App])
	Container -.- HeadFA([Header Feature App])
	Container -.- HomeFA([Homepage Feature App])
	Container -.- SearchFA([Search Feature App])
	Container -.- PLPFA([PLP Feature App])
	Container -.- PDPFA([PDP Feature App])
	Container -.- CartFA([Cart Feature App])
	Container -.- CheckoutFA([Checkout Feature App])
	Container -.- CustFA([Customer Feature App])
	FootFA -.- FootFS([Foot Feature Service])
	HeadFA -.- HeadFS([Head Feature Service])
	HomeFA -.- HomeFS([Home Feature Service])
	SearchFA -.- SearchFS([Search Feature Service])
	PLPFA -.- PLPFS([PLP Feature Service])
	PDPFA -.- PDPFS([PDP Feature Service])
	CartFA -.- CartFS([Cart Feature Service])
	CheckoutFA -.- CheckoutFS([Checkout Feature Service])
	CustFA -.- CustFS([Customer Feature Service])
	BFF([BFF GraphQL Server])
	FootFS<-->BFF
	HeadFS<-->BFF
	HomeFS<-->BFF
	SearchFS<-->BFF
	PLPFS<-->BFF
	PDPFS<-->BFF
	CartFS<-->BFF
	CheckoutFS<-->BFF
	CustFS<-->BFF
	BFF([BFF GraphQL Server]) <--> CustDS([Customer Domain Service])
	CustDS <--> Gigya([SAP CDC])
	BFF <--> ContDS([Content Domain Service])
	ContDS <--> CS([Content Stack])
	BFF <--> ProdDS([Product Domain Service])
	ProdDS <--> CT([Commerce Tools])
	BFF <--> SrchDS([Search Domain Service])
	SrchDS <--> AT([Attraqt])
	BFF <--> CartDS([Cart Domain Service])
	CartDS <--> CT
```