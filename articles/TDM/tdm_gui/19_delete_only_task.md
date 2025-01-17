# TDM Delete Task

A Delete task contains only the **Delete** task type and deletes (cleans) the selected entities from the target environment. The delete task also releases the cleaned entities if the entity is reserved.

Note that the deleted flows must be implemented in the Fabric implementation. 

Click for more information about the [delete implementation](/articles/TDM/tdm_implementation/08_tdm_implement_delete_of_entities.md).

A Delete task contains the following tabs:

- [General](14a_task_general_tab.md)
- [Additional Execution Parameters](#additional-execution-parameters-tab)
- [Requested Entities](#requested-entities-tab)
- [Task Scheduling](22_task_execution_timing_tab.md)

When checking the **Set Task Variables** setting, a new [Task Variables](23_task_globals_tab.md) tab opens.

## Additional Execution Parameters Tab

### Data Type

Check the **Entities** to delete the requested entities from the target environment.

### Additional Execution Parameters

#### Set Task Variables 

Check to open the Task Variables tab and [set the variable value on a task level](23_task_globals_tab.md).

### Post Execution Processes

Select all, partial, or one [post execution process](04_tdm_gui_business_entity_window.md#post-execution-processes-tab) of the selected BE.



## Requested Entities Tab

This tab defines the subset of entities for the task.

The following selection methods are available on load tasks: 

### Entity list 

This is the **default option**. Populate the list of entities for the task, separating them with a comma.  Note that a warning is given if the entity list has entities that are reserved for another user.

### Custom Logic

select a [Broadway flow](/articles/TDM/tdm_implementation/11_tdm_implementation_using_generic_flows.md#step-7---optional---build-broadway-flows-for-the-custom-logic--selection-method) in order to both get the entity list for the task and set the maximum number of entities for the task.

Notes:

-  TDM 8.0 added integration of [Broadway editors](/articles/TDM/tdm_implementation/15_tdm_integrating_the_tdm_portal_with_broadway_editors.md) into the TDM portal when populating the Custom logic parameters in the task’s tabs.
-  The **Filter out Reserved Entities** checkbox indicates if entities that are reserved for other users must be filtered out from the task's entity list. If checked, these entities are filtered out from the task's entity list.



 [![Previous](/articles/images/Previous.png)](18_load_task_data_versioning_mode.md)[<img align="right" width="60" height="54" src="/articles/images/Next.png">](20_reserve_only_task.md)
