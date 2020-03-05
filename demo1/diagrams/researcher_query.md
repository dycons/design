For use in: http://www.plantuml.com/plantuml/uml/rLB1ZjCm43tZhnZjgIjjnRrIqLOj8BHKj1iaBboyphGrSUnWJrgLhsViZKlJKB52ui0bhJNllNdpPku3IKzjROHKv3nu32Yzsg4N3HUqqesq13SFU5J6oGf67yhLxGt800_pFcJTeZ_2Uqbua4Tu3L-ShpR67M2sHKk9GiUgprdeG5u_jOGbp8tKTO2bj7AB7aQVZnMiJBlLzZQJ6txs1HxVBejKzLY72sr9CY310etUHsi5-hrMcE1bUO7-j9gbWcka4DCfp5hQeUMw9EqiJAT2_Ce6nczuV9X0qJxLci8bMjkDytNwBnWtPlWPUS8FXUnfm2DNeKtjKvYTF64vx1_ZE3FmFU4FcdOWMJChCOYnEJh6iw-3z6N8W5n93ZdzILBIR2tQ8eCr_uOad2a9D-wfFc9ed2qxNDa5Rt2F1bHQwwbJPfvPwH3PBAzc0g4tbxJb2IKDGQ7jrc1iwrgly3aj8PXv4bXtOnYd99jmDiH1CYQkcNs_onOySgD0C8eBkVjgksvtEOtRQ26hf065Y18d9TwAghfcZJDQCKa8WEYMJmj6gyMAHPh6kprsOyKwG9B9hdy7E9-gGHylQrZFeVh19X8e5Y_Ey1eMJS0ponw_eQAhZdFzXOPQ6w_JUVq3onpwomnUvw8jnIkHR2zRGtfm5W1_kVH8xMrvVdtB2N5rlqc0-HubSRbOVupDMhQyyQxEJjAHdNHX67R1ipwpKuDE8_pfJb9-6VmFZYPV3NlzJ-v5YFYzvqzRcvy0


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
