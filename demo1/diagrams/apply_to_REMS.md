For use in: http://www.plantuml.com/plantuml/uml/dL9TQnDD5BwVN_6GYvVNYv3Ioq2qCNebMcdqIgKSPawQiPkPPSxPI5hwtvsFhas5e0ZBidi-dezdvaAKaWwxpf3Lc31PzdtdBGMTW0GqbjyiLxNWo9e-RtZh-po4oTNRz-lQIRoJvwrL3C911MjEUs-vZbanPh705GjJibjEeCB8Watc4ROnE4e73CA86pXI4jR_vGKM2sC8qvULQOwFb-jFc57lPzGagyzyCvkObF4rl8xkCCTzhbCNRmQUUkNTr3jvC0NW2MZLDWY1V-08wt4ipz4SvsTdk7mtmjuVZMN8XsefHuGTtRA0gknfiK23RcFoTtKKZgsNyf4V2iTapLsd5INnTxOgq2q_4uGI_nyJQDidfQ9kezdw4DLl3YNejjLOHkE5RhHkk1EkwiXQxlzPNVv9SSz_8Szca2_zeE0z1yKsnjjlUQtXPf2Yzze-th6P3ruzTVxk5_Ink6ll360pxgWlsph7ibhXqWjH6SdVRDK0GBdMS96bD11gJBLH4lXlX0uAlwdbt-ywEAMOv5LkSd0LlGeOZNqMXkCD3PrYz02DEBBHB3wrJvrSIvIwamSNdBd8hs7N_GG0

@startuml
actor "Applicant" as a
actor "DAC" as dac
participant "REMS" as rems
participant "Consents Service" as cs

== Researcher applies for secondary use of dataset(s) ==

a -> rems: POST /api/applications/create \nBody: {catalogue-item-ids}
rems -> rems: Create a new application
rems --> a: 200 OK \nBody:{application-id}

== DAC makes a data-use authorization decision on an application ==

alt DAC rejects the application
  dac -> rems: POST /api/applications/reject \nBody:{application-id}
  rems -> a: Notify of rejection
  rems --> dac: 200 OK
else DAC accepts the application
  dac -> rems: POST /api/applications/approve \nBody:{application-id}
  rems -> cs: [put event hook] \nPOST /project_consents/initialize \nBody: application_id
  cs --> rems: 202 Accepted
  note right
    REMS does not notify
    DAC & applicant if
    Consents Service errors?
  end note
  rems -> a: Notify of approval
  rems --> dac: 200 OK
  cs -> cs: Use default consents to initialize project consents
end
@enduml