For use in:
http://www.plantuml.com/plantuml/uml/rLNDRXCn43xZfnZrb4gjk5SAQjK2GYKYwI9nuU9w9ubPhxtOiud2qpDsRPRDqbH2ui35IVJzp3S_Kgu3IKzzQuHKv3nu32Yzsg8N7GDQwKvQWhDxF2ZZP0Ep3_No_0naWFlRCMNLep_0UqcubmTu3L-SRzR6xU5cWvQIX0xDM7Ed0tdzs1FMS6kaRWCif9KRz33xV6XWQVgDjkT5tF7FiS73sqMLK8zjEFAM90CItA7HknYjGh_J2XEzhe_OdvOt2N2T568-XvkUjcWPbwITXVbSY64sSFc6suShgFh7LbCuPckxThvDtmNpBfa-Y5dcKI5dckZ8DMXJ-JJCR4zOJ-l7SVmMu7lYxpLjGR9dTc8EKylacCwnzjBN8UC5IFdQm8JeepbO34K6M_uCqROu9jhboFC1G-TidMbrdhh17HfGQQUbJX4EEJAX1EMjN44OjIHjNKX26e9Asut1MDJrNk7Je8I0gFrbiSfT6ASaXt0sn3u64Y-PWxyh8JnohK0mY-ku-6fpmOioZVEOejYa3WL4Zy6AbwAQNAvpHYjMnL0nCLyMp9PtItRfEkzsc5jZEBkKs5Dyr5M7FfxL12v3zPD590vZHnVuHofpqGGSuZBf6ofwBkq4Ffccj_4idTS_i2xpFfNkPIdQMhmIAVAYDwGd9_7NspvUxks9vJBZcRLcFSizIcAqMdsCfNeElFQkBJdIRHNRwPGhxZDfdzopGtZXEDBa2lTVNGhd_0zkHCJdcf_u9_SJ


@startuml
actor researcher
participant "Keycloak (IdP)" as KC
participant "Query Gateway / Workflow Agent" as tyk
participant "Distributed Claims Manager (Tyk Plugin)" as tykplug
participant "REMS" as rems
participant "Dataset Delivery Service" as dds
participant "OPA" as opa

== Authentication ==

researcher -> KC : Submits Authentication information
KC --> researcher : receives Authentication token

== Query Submitted ==

researcher -> tyk : Submits query with auth token
tyk -> tykplug : forwards token
tykplug -> rems : queries entitlements with token
rems --> tykplug : Responds with dataset level claims
tykplug --> tyk: forwards claims
tyk -> dds: submits query and claims

alt Single Resource Query

  dds -> dds: Queries local consents
  dds -> opa : Forwards request (Auth,\n claims, queries) and consent data
  opa -> opa : check local policies

  alt OPA (PDP) approves query
    opa --> dds: approves query
    dds -> dds: performs query
    dds --> tyk: responds with data
    tyk --> researcher: responds with data

  else OPA (PDP) rejects query
    opa --> dds: rejects query
    dds --> tyk: responds with unauthorized request
    tyk --> researcher: responds with unauthorized request
  end
	
else Multi Resource Query

  dds -> opa : Forwards request (Auth,\n claims, queries)
  opa -> opa : Check local policies

  alt opa approves query
    opa -> opa: Creates SQL clause from request and policies
    opa --> dds: Sends SQL query clauses
    dds -> dds: performs query with clauses
    dds --> tyk: responds with data
    tyk --> researcher: responds with data

  else opa rejects query
    opa --> dds: rejects query
    dds --> tyk: responds with unauthorized request
    tyk --> researcher: responds with unauthorized request
  end

end
@enduml
