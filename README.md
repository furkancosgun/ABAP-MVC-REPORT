# ABAP-MVC-REPORT
### ABAP MVC REPORT TEMPLATE

### Model
```abap
*&---------------------------------------------------------------------*
*&  Include           ZFC_EXAMPLE_MODEL
*&---------------------------------------------------------------------*
CLASS cls_model DEFINITION.
  PUBLIC SECTION.
    TYPES BEGIN OF mty_table.
    INCLUDE TYPE scarr.
    TYPES END OF mty_table.


    METHODS :
      select_data ,
      get_output_data RETURNING VALUE(rr_data) TYPE REF TO data.

  PRIVATE SECTION.
    DATA mt_output TYPE STANDARD TABLE OF mty_table.
ENDCLASS.

CLASS cls_model IMPLEMENTATION.
  METHOD get_output_data.
    rr_data = REF #( mt_output ).
  ENDMETHOD.

  METHOD select_data.
    SELECT * FROM scarr
    INTO CORRESPONDING FIELDS OF TABLE @mt_output.
  ENDMETHOD.
ENDCLASS.
```

### View
```abap
*&---------------------------------------------------------------------*
*&  Include           ZFC_EXAMPLE_VIEW
*&---------------------------------------------------------------------*
CLASS cls_view DEFINITION.
  PUBLIC SECTION.
    METHODS :
      display,
      get_model RETURNING VALUE(ro_result) TYPE REF TO cls_model,
      set_model IMPORTING io_model TYPE REF TO cls_model,
      prepare_display.

  PRIVATE SECTION.
    CONSTANTS:mc_strcutre_name TYPE dd02l-tabname VALUE 'SCARR'.
    DATA :mo_model      TYPE REF TO cls_model,
          mo_alv        TYPE REF TO cl_gui_alv_grid,
          mo_controller TYPE REF TO cls_controller,
          ms_variant    TYPE disvariant,
          ms_layout     TYPE lvc_s_layo,
          mt_fieldcat   TYPE lvc_t_fcat.
ENDCLASS.

CLASS cls_view IMPLEMENTATION.
  METHOD get_model.
    ro_result = mo_model.
  ENDMETHOD.

  METHOD set_model.
    mo_model = io_model.
  ENDMETHOD.
  METHOD prepare_display.
    CLEAR ms_layout.
    ms_layout-zebra      = 'X'.
    ms_layout-col_opt    = 'X'.
    ms_layout-cwidth_opt = 'X'.
    ms_layout-sel_mode   = 'A'.
    ms_variant-report = sy-repid.

    REFRESH  mt_fieldcat.
    CALL FUNCTION 'LVC_FIELDCATALOG_MERGE'
      EXPORTING
        i_structure_name = mc_strcutre_name
      CHANGING
        ct_fieldcat      = mt_fieldcat
      EXCEPTIONS
        OTHERS           = 0.

  ENDMETHOD.
  METHOD display.
    FIELD-SYMBOLS <lt_table> TYPE STANDARD TABLE.

    DATA(lr_data) = mo_model->get_output_data( ).
    ASSIGN lr_data->* TO <lt_table>.

    mo_alv = NEW cl_gui_alv_grid(
                  i_parent = cl_gui_container=>default_screen
                  i_appl_events = abap_true ).

    mo_controller = NEW cls_controller( ).
    mo_controller->mo_model = me->get_model( ).
    mo_controller->mo_grid = mo_alv.
    SET HANDLER mo_controller->handle_double_click FOR mo_alv.

    mo_alv->set_table_for_first_display(
                EXPORTING
                  i_bypassing_buffer = abap_true
                  is_variant         = ms_variant
                  i_save             = 'A'
                  i_default          = 'X'
                  is_layout          = ms_layout
                CHANGING
                  it_fieldcatalog    = mt_fieldcat
                  it_outtab          = <lt_table> ).

    WRITE: space.
  ENDMETHOD.

ENDCLASS.
```

### Controller
```abap
*&---------------------------------------------------------------------*
*&  Include           ZFC_EXAMPLE_CONTROLLER
*&---------------------------------------------------------------------*

CLASS cls_controller DEFINITION.
  PUBLIC SECTION.
    METHODS:
      handle_double_click FOR EVENT double_click OF cl_gui_alv_grid IMPORTING e_row e_column.
    DATA mo_view TYPE REF TO cls_view.
    DATA mo_model TYPE REF TO cls_model.
    DATA mo_grid TYPE REF TO cl_gui_alv_grid.
ENDCLASS.

CLASS cls_controller IMPLEMENTATION.
  METHOD handle_double_click.
    FIELD-SYMBOLS :<lt_table> TYPE STANDARD TABLE,
                   <fs_tab>   TYPE cls_model=>mty_table.
    DATA(lr_data) = mo_model->get_output_data( ).
    ASSIGN lr_data->* TO <lt_table>.

    DELETE <lt_table> INDEX 1.
    mo_grid->refresh_table_display( ).
  ENDMETHOD.
ENDCLASS.
```

### Base / Main
```abap
*&---------------------------------------------------------------------*
*& Report ZFC_EXAMPLE_MVC
*&---------------------------------------------------------------------*
*&
*&---------------------------------------------------------------------*
REPORT zfc_example_mvc.
CLASS :
    cls_model      DEFINITION DEFERRED,
    cls_view       DEFINITION DEFERRED,
    cls_controller DEFINITION DEFERRED.

INCLUDE :
      zfc_example_model,
      zfc_example_controller,
      zfc_example_view.

DATA: lo_view  TYPE REF TO cls_view,
      lo_model TYPE REF TO cls_model.

INITIALIZATION.
  CREATE OBJECT lo_model.
  CREATE OBJECT lo_view.

START-OF-SELECTION.
  lo_model->select_data( ).
  lo_view->set_model( lo_model ).

END-OF-SELECTION.
  lo_view->prepare_display( ).
  lo_view->display( ).
```
