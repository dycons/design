For use in:
http://www.plantuml.com/plantuml/uml/ZP9FRnCn5CJl_XIZd44RK4-HWbhA7o7-D2BDhLBqMB-qXbrxPR-bf8JlZjVOqIO4qKDoySnlFECz3q9U-hPHL8lpc2oyQSblyOg4i1I-9wyde80kQDL5hQbDHrOmkUYwnjGanbaisNytDOUaf70axENk0HzPI0_GYrksyaVOqg7oqjaG3qFX9LKTb9hFySuQCTIO1mR1yZt6pYzRr9v9ZBs5tB7nqBoXlX7K1JcVAnKxuEcBI9nXSRMyHjMv8EiHIDMX22u77X-aDCP-OmrnGqRWlpAIEzfZp0pFfyyoG-F6hZv_cA4QmKBrCqYljskZezLi3FjVGy_igLQT7kBJOXldGUERZCjGBhDRh6AXfB-nBvtUpfBv2KxaK4ZEbieEVKfRboAdxXRLPPgthw__swNglFlAjUoPIb4ZM8nAy0yJ9C1O7B-x0lAKCNMOSeghz_jQcOmz6OCCHPsNAwTeyuTNP6cp0bL07YC_UTbyLpZ5-e-dtHKGDu4FwbadsBZUwaSqF9kUuLFKlndqHnSzFXlVvF2Cu-yy_lLjyni0

@startuml
actor "Primary Data Steward" as psd
participant "Application Services" as as
participant "Authz Metadata Agent" as ama
participant "Consents Service" as cs

== Initialize a new participant by creating their default consents ==

psd -> cs: POST /post_participant
cs -> cs: Create participant linked to these default consents
cs --> psd: 201 Created \nURL: /participants/{study_identifier}

== Create/Update data for a participant ==

psd -> as: POST|PUT /data \nBody: data, {study_identifier}
as -> ama: POST /update_consents/{study_identifier}
ama -> cs: GET /participants/{study_identifier}/project_consents

alt Participant exists in Consents Service
  cs --> ama: 200 OK \nBody: project consents
  ama -> ama: Update consents metadata
  ama --> as: 200 OK
  as -> as: Create data | Update data
  as --> psd: 201 Created | 200 OK
else Participant not found in Consents Service
  cs --> ama: 404 Not Found
  ama --> as: 404 Not Found
  as --> psd: 404 Not Found
end
@enduml
