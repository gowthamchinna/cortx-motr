```plantuml
@startuml

title Simple Object Upload in LR R2

participant "S3 Client" as client
participant "S3 Server/N1" as N1
participant "ctrl-A/CL-1" as CL_1
participant "ctrl-B/CR-1" as CR_1
box Node2
participant "N2" as N2
participant "CL-2" as CL_2
participant "CR-2" as CR_2
end box
box Node3
participant "N3" as N3
participant "CL-3" as CL_3
participant "CR-3" as CR_3
end box
box Node4
participant "N4" as N4
participant "CL-4" as CL_4
participant "CR-4" as CR_4
end box
box Node5
participant "N5" as N5
participant "CL-5" as CL_5
participant "CR-5" as CR_5
end box
box Node6
participant "N6" as N6
participant "CL-6" as CL_6
participant "CR-6" as CR_6
end box

autoactivate on

client -> N1: PUT /bucket_name/object_name

N1 -> N2: get_keyval(global_bucket_index,\n key = "bucket_name")
N2 --> N1: value = account_id of bucket owner

N1 -> N3: get_keyval(global_bucket_md_index,\n key = "account_id/bucket_name")
N3 --> N1: value = bucket metadata JSON

N1 -> N5: get_keyval(BUCKET_nnn_obj_index,\n key = "object_name");
note left
   * Index OID for this index is stored
      in bucket metadata JSON.
   * This call checks if object is present.
end note
N5 --> N1: not found

N1 -> N1: create_object
N1 --> N1: success (completed)

loop until all data is written
  N1 -> N4: Write data
  N1 -> N5: Write data
  N1 -> N6: Write data

  N5 --> N1: success
  N6 --> N1: success
  N4 --> N1: success
end

N1 -> N6: put_keyval(BUCKET_nnn_obj_index,\n key = object_name, val = object_metadata)
N6 --> N1: success (completed)

N1 --> client: 200 OK

@enduml