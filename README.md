# Modbus New

This introduces the following changes, compared to the Home Assistant Modbus integration:

* Added map attribute
  - The map attribute introduces the capability to translate numerical values into meaningful strings. This is especially useful when sensor values correspond to specific states or conditions that are better represented as text. Utilizing the device_class: enum allows the system to recognize and handle these values as a set of enumerated states.
    ```
    - name: Dimplex Heatpump Status Message
        address: 103
        data_type: uint16
        device_class: enum
        map:
            '0': 'Off'
            '1': 'On'
            '2': 'Heating'
    ```
* Added sum_scale attribute
  - The sum_scale attribute is used to differently scale multiple readings before summing them. The following example shows its use with something like `data_type: custom`, `count: 3`, and `structure: ">HHH"`. But instead of returning the values as comma separated list, (in this case) three readings from the heat pump are scaled differently before being summed to provide a total heat supplied value in kWh.
    ```
    - name: Dimplex Heatpump Heat Supplied
        address: 5096
        data_type: uint16
        precision: 0
        sum_scale: [1, 10000, 100000000]
        unit_of_measurement: kWh
        state_class: total_increasing
    ```
* Only warn when duplicates are configured
  - The integration has been adjusted to only issue warnings for configurations that include duplicate entities rather than blocking them. This change allows for greater flexibility, particularly when monitoring the same device status parameter across multiple climate sensors is necessary.
* Added target_temp_scale + target_temp_offset attributes
  - Previously, the same scale was applied for the `target_temp_register` and the `address` register. Some devices need different scaling, which can now be done with `target_temp_register` and `target_temp_offset`. If `target_temp_register` or `target_temp_offset` are not defined, `scale` and `offset` are used like before for both the `target_temp_register` and the `address` register.
    ```
    - name: Dimplex Heatpump Hot Water
        address: 3 # Dimplex heatpump water temperature
        target_temp_register: 5047 # Dimplex heatpump target water temperature
        target_temp_write_registers: false
        target_temp_scale: 1
        hvac_mode_register:
          address: 103 # Dimplex heatpump status register
          values:
            state_off: [0, 1, 2, 3, 5, 24]
            state_heat: 4
        input_type: holding
        data_type: int16
    ```
