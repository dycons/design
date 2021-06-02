For use in:
http://www.plantuml.com/plantuml/png/pLLDRnit43sNlsAGIrl0Gbeq9m4kugIEuzPkbCX0S-Xbj1n9hBWao-76TNzzXn-ALh8ZzLJqeiFopPitZs-ebvmmfg6t2hla8qHYmjYjAQeW1wOp0Ls2Pny-VuUPZmdjCq266FPHlz13Pprku9jhFVksG3RT0T7bxRpMKiy7rPXuW8lPTIt4O9uy7zBQH_Ct9kCTt1gjBTrZf5hKFrNo2hiDELqHYtt4kzb5-Uu37WgzUYUpMlb0t1Ill5BdvqK2EO5M3UVdIkryWycFuWjCuTgPP30H0nEp09LydiZf23kL3fmORlpAE5WCANdt5DThaR4vX8LylEda-0Df4wbBa3pWJXp1m1ILG8vrvIuIxNVTyiw7HMyI5ng7FPr1GEPx7tKXhUhssyyfVZ8TmIpwJqOBsRr9wo8AVlxjxeW5RtrFJGlyxZhl4XfdtAgDcxaNqTypGRdKSX5lA7NhZCcf0gdLUNAqdE_aY8yxobTZokfe3gP0tmyK7-0v1lCyKEnDkK86v6fgHkqSkBfuSVKMPiWSV4mmKezvxFs1gbYBfkSyHUjKlKwag_PwkTAzIJ74S7LvLmKPfusuEyYaGbqS_ZXvVV7hJzTNaykFTvFIOKseAOwrxAjKXJrtwQVmnaTP8itmPn5pb58vdmWiBTFqP5iUNVNNMQpaueyS5LmkZJLrWOm3oImiVUp1Bm5p0304snPiOhHIcxv8NUpSiBv3c_imkSIvMbWCmq7bPb7ptEUSfY6wmqgbq2QOYq6M1C9-Y1tLIr40jQyF22EJ8ebDdCv4vvSCE9CnzBO7Q4oe8DFKXAry3kEQkaqJ4zf3erI3rQPJM96ZY2dFQpY9FTKRsIxIXPPfDtpBPcDeKz-qkaTLWjkwEfhD7eCljl1Rt50BYtmV1I9k3LoGE5eGXdGMAC9xVCXOeBzCTKzQz27LFNdntVUvEcJBnCH8sGgqwVC0ybHF7ezkk58AJvPIHRRTcJP2pJNHxHgW7BRj_IBcqLFLNggSrPtGfKomVP40o3ATrlklj0uEspyTwMtWJcWN2t_85xamMfCxEiG-VK_CkA8yWDFgArN6k1riCeSByTzlnFzu8KOR0MDXy_St-NsNrWp9ooHXi7BGdkh7H3L44fUz5GiKyukuzoGtLdKwvQSphjIHU3oPxnoEb_8pzFOV


@startuml
actor researcher
participant "Researcher Portal" as rp
participant "Keycloak (IdP)" as kc
participant "REMS" as rems
participant "Katsu API" as kapi
participant "Katsu Authorization Middleware" as km
participant "Katsu Backend" as kb
participant "OPA" as opa
participant "REGO Policies" as rego

== Authentication ==

researcher -> rp : Initiates session
rp --> researcher : Posts Login button
researcher -> rp : Clicks Login button
rp -> kc : Redirect to authenticate user
kc -> researcher : Posts Login screen
researcher --> kc : Submits username, password
kc --> rp : Redirect to Service Provider with auth JWT
rp --> researcher : Posts Home screen \ncontaining button to browse Katsu

== Fetching REMS credentials ==

researcher -> rp : Clicks button to Browse Katsu
rp -> rems : Query /api/permissions as user
rems --> rp : GA4GH Passport containing a JWT \ncontaining REMS claims

== Query Submitted ==

rp -> kapi : Queries GET /api/individuals \nwith X-CANDIG-EXT-REMS header containing GA4GH Passport
kapi -> km: Forwards query with X-CANDIG-EXT-REMS header
note left: X-CANDIG-EXT-REMS header contains researcher's \nProject affiliation in the form of an \napplication-id

km -> kb: Queries local consents for this application-id
kb --> km: Returns local consents

alt Single Resource Query
  km -> opa : Forwards request, \nX-CANDIG-EXT-REMS header, and consent data
  opa -> rego : Check local policies


  opa -> opa: generates list of \napproved datasets for this user
  opa --> km: List of approved datasets
  km -> kb: Forwards list of approved datasets
  kb -> kb: Makes query
  note left: resource being queried for \nmay not exist, \nprompting a 401 response here

  alt List of OPA-approved datasets is not empty
    kb --> kapi: responds with data
    kapi --> rp: responds with data
    rp --> researcher: Posts the response from Katsu

  else List of OPA-approved datasets is empty
    kb --> kapi: responds with unauthorized request
    kapi --> rp: responds with 403 Forbidden
    rp --> researcher: Posts unauthorized message
  end
	
else Multi Resource Query

  km -> opa : Forwards request, \nX-CANDIG-EXT-REMS header, and consent data
  opa -> rego : Check local policies

  opa -> opa: generates list of \napproved datasets for this user
  opa --> km: List of approved datasets
  km -> kb: Forwards list of approved datasets

  kb -> kb : generates SQL clauses to populate \nwith approved datasets from OPA
  kb -> kb: performs query with clauses
  kb --> kapi: responds with data
  kapi --> rp: responds with data


  rp --> researcher: Posts response from Katsu

end
@enduml