For use in:
http://www.plantuml.com/plantuml/uml/pLJ1RXit43sNNp78fIrW8IsQam0NSLDRSUiqIcIWEVGoMeuaLhiao-76TR--CqkgAycEd5jpiOMTuPlttZpo9MTCkUzQWqqE2H8nOMesb4oKWcjSH9_XsVl_liCyf8pjCq2650-xVgNxfWsuXM-kxVpReMNR705TlbcKizJnqMdCtCDL2ZwJj-Nqwf6MKj5VXyMtyukX528Upvoz5TgjV22kmzV1cFDpkEZpXhoqOdR1m_cvCQC5CFbH9C8KhhtB3ZCnC35Bev7F4wsElLEPm9dX9goTXsMVS_2FORqI6jlZjgZbRIFbzsHTeYP33e0ZEOAUAHbfEJMLcqHqE7M7k-YNdSiym7Ziw0mYCj-5P0VGokvW_8BIHzSGp5FuwAo0tRcy6KZ1Bx_Vdh3WTUYeSe4_V1DyHkUTNrUvYhrCuOu9XdqEYxYYt6oqHmC2SjQgScnPTtB2nn7ofpt8wgXcJ5hVzPJkuJb6zpnIwjomGWRaOgfsZGxCBbxCNiCScMD86VROeyeU1OraMtGTgueogNgToGxSwt9NbOWZXjdbRQ7Zl7NYRIyw1NZmzyFarSLlFr_F9fSVRYS3_ePGtH_77d4qWgqJLFWKhaAwmsGP_XgudCGqFcI2bbPv-kHStjlqryfMOl6d9WLNAzUwSd-S1uaih4Bg8Am0TVyOOrllryHPOx3Di10LBKdQEFIfeUARCL14xAdPETQ61bkbmkGpwm0Pv1WEmVS1GaJOSoEH-C7vJ0XzIkcPyBMx6M0nem65AKbQ1mtTXffj9HFhWsBCs2R_fh0cJmcpidQSnOTYWjmYiWCiqoYXPh0YZC8NvEq8yE3u8rP5IkfxNRmqx2qtxOnUaZW_T3Yooa6uTNYllK1_ksAGpEfYBf5_yTttUZggA-9J8bMhAxofrEJLdJnKvxX02evIgMoBmDMq8bekyQnMWvJgFVnanuD7fxuv6hkHvqe4xDuM06gPdgJxcLHxh-zVIEuVihjCdQGk1lwWALmwQycVrd00tX4phadvUsk-CeEADtsRtV6r-LzI_qM47lRdBjxTw5Cio0mv2D7OjrAex-f3j98IoSEJTwaiicARpyhowIGRSo85ZwPOC_1I_lHT-oy0


@startuml
actor researcher
participant "Researcher Portal" as rp
participant "Keycloak (IdP)" as kc
participant "REMS" as rems
participant "Katsu Frontend" as kf
participant "Katsu" as katsu
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

rp -> kf : Queries GET /api/individuals \nwith X-CANDIG-EXT-REMS header containing GA4GH Passport
kf -> katsu: Forwards query with X-CANDIG-EXT-REMS header
note left: X-CANDIG-EXT-REMS header contains researcher's \nProject affiliation in the form of an \napplication-id

alt Single Resource Query

  katsu -> katsu: Queries local consents for this application-id
  katsu -> opa : Forwards request, \nX-CANDIG-EXT-REMS header, and consent data
  opa -> rego : Check local policies


  opa -> opa: generates list of \napproved datasets for this user
  opa --> katsu: Sends list of approved datasets
  katsu -> katsu: performs query
  note left: resource being queried for \nmay not exist, \nprompting a 401 response here

  alt List of OPA-approved datasets is not empty
    katsu --> kf: responds with data
    kf --> rp: responds with data
    rp --> researcher: Posts the response from Katsu

  else List of OPA-approved datasets is empty
    katsu --> kf: responds with unauthorized request
    kf --> rp: responds with 403 Forbidden
    rp --> researcher: Posts unauthorized message
  end
	
else Multi Resource Query

  katsu -> opa : Forwards request, \nX-CANDIG-EXT-REMS header, and consent data
  opa -> rego : Check local policies

  opa -> opa: generates list of \napproved datasets for this user
  opa --> katsu: Sends list of approved datasets
  katsu -> katsu : generates SQL clauses to populate \nwith approved datasets from OPA
  katsu -> katsu: performs query with clauses
  katsu --> kf: responds with data
  kf --> rp: responds with data


  rp --> researcher: Posts response from Katsu

end
@enduml