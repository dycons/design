For use in:
http://www.plantuml.com/plantuml/uml/dPDBRzim3CVl_XH43yDsADBr621TktQdFLhqSTgAW92OMgiZ6IAT8ItwtKV9TXvPqcm3aY0soJz__5CwPS4epRGgq4Y8S71MrvKpwEK0a07xrnTdv_c5HLFKMk6CgpK93gu_VRt9aKXJNWzTe21Sk4X9OWut56VEK2jZDtBFWsVomcjPXejYD89hOi9eIeg0YOuO9ih4P8AtgCeD4uI9iWaooLj-1wDHKI0SdcIg8LnTtjp206kdtpoUEEqtC95G27xuZy7EX_3zMGDOXOU6naveEdRsvRv82akbyxO4mTClM17hqZGFXt1yT0INdtlTvpg6dsHal9AUfFDiwYjCyP4OCFCVfbcmaJ94zvHJmP9ndFxe1_rgqtRCIh9AfDI4GKfQ0uEqixrMjDLxaI6hG3lcjo1kCa_Ev-gksy88RTjRKL35b2dH6Ah_cr8VOfZHFs1QoxhOgxjRe1bvWJA4ntkjJmU8q9ImPUT3ZRS0tOdYmPQiWVDE75RkYSOzX-bFSKy8ho1QguJzn6hKCPnbSyXgq0SXYEwX57q0IBSCR21E0V3P-1J8dhxfdVG2Rf9VRrude1X3P8sHjrbyrofRYx7QjSaU_0ICKYzjP6diN2SZhqgEuWiudxVSURWqTUNUy-2FkjTy_HjNpOBjZWaiJR2f11QBaG1BVMXbbk6klZitElBxxi9RRNrt_MMudtxpA7pOZZhQEG9vMvpgJpEjVWC0

@startuml
actor "Applicant" as a
actor "DAC" as dac
participant "REMS" as rems
participant "Data Directory Service" as dds
participant "Consents Service" as cs

== Researcher applies for secondary use of dataset(s) ==

a -> rems: POST /api/applications/create \nBody: [{catalogue_item_id}]
rems -> rems: Create a new application
rems --> a: 200 OK \nBody:{project_application_id}

== DAC makes a data-use authorization decision on an application ==

alt DAC rejects the application
  dac -> rems: POST /api/applications/reject \nBody:{project_application_id}
  rems -> a: Notify of rejection
  rems --> dac: 200 OK
else DAC accepts the application
  dac -> rems: POST /api/applications/approve \nBody:{project_application_id}
  rems -> dds: [PUT event hook] \nfor each {catalogue_item_id}: \nPOST /datasets/{catalogue_item_id}/initialize_project_consents  \nBody: {project_application_id}
  dds --> rems: 202 Accepted
  note right
    REMS does not notify
    DAC & applicant if
    Consents Service errors
  end note
  rems -> a: Notify of approval
  rems --> dac: 200 OK

  dds -> cs: for each {study_identifier} in the dataset: \nPOST /participants/{study_identifier}/initialize_project_consent \nBody: {project_application_id}
  cs -> cs: Use default consents to initialize project consents
  cs --> dds: 201 Created \nURL: /participants/{study_identifier}/project_consents?project_application_id={project_application_id}
end
@enduml