import "engine_sim.mr"

units units()
constants constants()
impulse_response_library ir_lib()

private node wires {
    output wire0: ignition_wire();
    output wire1: ignition_wire();
    output wire2: ignition_wire();
    output wire3: ignition_wire();
    output wire4: ignition_wire();
    output wire5: ignition_wire();
}

private node M4_head {
    input intake_camshaft;
    input exhaust_camshaft;
    input chamber_volume: 50 * units.cc;
    input intake_runner_volume: 149.6 * units.cc;
    input intake_runner_cross_section_area: 1.75 * units.inch * 1.75 * units.inch;
    input exhaust_runner_volume: 50 * units.cc;
    input exhaust_runner_cross_section_area: 1.75 * units.inch * 1.75 * units.inch;

    input flow_attenuation: 1.0;
    input lift_scale: 1.0;
    input flip_display: false;
    alias output __out: head;

    function intake_flow(50 * units.thou)
    intake_flow
    .add_flow_sample(0 * lift_scale, 0 * flow_attenuation)
    .add_flow_sample(50 * lift_scale, 58 * flow_attenuation)
    .add_flow_sample(100 * lift_scale, 103 * flow_attenuation)
    .add_flow_sample(150 * lift_scale, 156 * flow_attenuation)
    .add_flow_sample(200 * lift_scale, 214 * flow_attenuation)
    .add_flow_sample(250 * lift_scale, 249 * flow_attenuation)
    .add_flow_sample(300 * lift_scale, 268 * flow_attenuation)
    .add_flow_sample(350 * lift_scale, 280 * flow_attenuation)
    .add_flow_sample(400 * lift_scale, 280 * flow_attenuation)
    .add_flow_sample(450 * lift_scale, 281 * flow_attenuation)

    function exhaust_flow(50 * units.thou)
    exhaust_flow
    .add_flow_sample(0 * lift_scale, 0 * flow_attenuation)
    .add_flow_sample(50 * lift_scale, 37 * flow_attenuation)
    .add_flow_sample(100 * lift_scale, 72 * flow_attenuation)
    .add_flow_sample(150 * lift_scale, 113 * flow_attenuation)
    .add_flow_sample(200 * lift_scale, 160 * flow_attenuation)
    .add_flow_sample(250 * lift_scale, 196 * flow_attenuation)
    .add_flow_sample(300 * lift_scale, 222 * flow_attenuation)
    .add_flow_sample(350 * lift_scale, 235 * flow_attenuation)
    .add_flow_sample(400 * lift_scale, 245 * flow_attenuation)
    .add_flow_sample(450 * lift_scale, 246 * flow_attenuation)

    generic_cylinder_head head(
        chamber_volume: chamber_volume,
        intake_runner_volume: intake_runner_volume,
        intake_runner_cross_section_area: intake_runner_cross_section_area,
        exhaust_runner_volume: exhaust_runner_volume,
        exhaust_runner_cross_section_area: exhaust_runner_cross_section_area,

        intake_port_flow: intake_flow,
        exhaust_port_flow: exhaust_flow,
        valvetrain: standard_valvetrain(
            intake_camshaft: intake_camshaft,
            exhaust_camshaft: exhaust_camshaft
        ),
        flip_display: flip_display
    )
}

private node M4_camshaft {
    input lobe_profile;
    input intake_lobe_profile: lobe_profile;
    input exhaust_lobe_profile: lobe_profile;
    input lobe_separation: 114 * units.deg;
    input intake_lobe_center: lobe_separation;
    input exhaust_lobe_center: lobe_separation;  
    input advance: 0 * units.deg; 
    input base_radius: 0.5 * units.inch;
    output intake_cam_0: _intake_cam_0;
    output exhaust_cam_0: _exhaust_cam_0;
    camshaft_parameters params (
        advance: advance,
        base_radius: base_radius
    )

    camshaft _intake_cam_0(params, lobe_profile: intake_lobe_profile)
    camshaft _exhaust_cam_0(params, lobe_profile: exhaust_lobe_profile)
    label rot360(360 * units.deg)
    _exhaust_cam_0
        .add_lobe(rot360 - exhaust_lobe_center + 0.0 * units.deg)
        .add_lobe(rot360 - exhaust_lobe_center + 480.0 * units.deg)
        .add_lobe(rot360 - exhaust_lobe_center + 240.0 * units.deg)
        .add_lobe(rot360 - exhaust_lobe_center + 600.0 * units.deg)
        .add_lobe(rot360 - exhaust_lobe_center + 120.0 * units.deg)
        .add_lobe(rot360 - exhaust_lobe_center + 360.0 * units.deg)
    _intake_cam_0
        .add_lobe(rot360 + exhaust_lobe_center + 0.0 * units.deg)
        .add_lobe(rot360 + exhaust_lobe_center + 480.0 * units.deg)
        .add_lobe(rot360 + exhaust_lobe_center + 240.0 * units.deg)
        .add_lobe(rot360 + exhaust_lobe_center + 600.0 * units.deg)
        .add_lobe(rot360 + exhaust_lobe_center + 120.0 * units.deg)
        .add_lobe(rot360 + exhaust_lobe_center + 360.0 * units.deg)
}

public node M4_engine {
    alias output __out: engine;

    engine engine(
        name: "BMW M4",
        starter_torque: 170 * units.lb_ft,
        starter_speed: 500 * units.rpm,
        dyno_min_speed: 1800 * units.rpm,
        redline: 15000 * units.rpm,
        throttle_gamma: 2,
        fuel: fuel(
            molecular_mass: 130 * units.g,
            energy_density: 53.1 * units.kJ / units.g,
            density: 0.755 * units.kg / units.L,
            molecular_afr: 12.5,
            max_burning_efficiency: 0.8,
            burning_efficiency_randomness: 0.5,
            low_efficiency_attenuation: 0.6,
            max_turbulence_effect: 2,
            max_dilution_effect: 10
        ),
        hf_gain: 0.01,
        noise: 1,
        jitter: 0.1,
        simulation_frequency: 10000
    )

    wires wires()

    label stroke(90 * units.mm)
    label bore(84 * units.mm)
    label rod_length(144.3 * units.mm)
    label rod_mass(50 * units.g)
    label compression_height(25.4 * units.mm)
    label crank_mass(10 * units.kg)
    label flywheel_mass(10 * units.kg)
    label flywheel_radius(100 * units.mm)

    label crank_moment(
        disk_moment_of_inertia(mass: crank_mass, radius: stroke)
    )
    label flywheel_moment(
        disk_moment_of_inertia(mass: flywheel_mass, radius: flywheel_radius)
    )
    label other_moment( // Moment from cams, pulleys, etc [estimated]
        disk_moment_of_inertia(mass: 1 * units.kg, radius: 1.0 * units.cm)
    )

    crankshaft c0(
        throw: stroke / 2,
        flywheel_mass: flywheel_mass,
        mass: crank_mass,
        friction_torque: 1.0 * units.lb_ft,
        moment_of_inertia:
            crank_moment + flywheel_moment + other_moment,
        position_x: 0.0,
        position_y: 0.0,
        tdc: 91 * units.deg
    )

    rod_journal rj0(angle: 0 * units.deg)
    rod_journal rj1(angle: 480.0 * units.deg)
    rod_journal rj2(angle: 240.0 * units.deg)
    rod_journal rj3(angle: 600.0 * units.deg)
    rod_journal rj4(angle: 120.0 * units.deg)
    rod_journal rj5(angle: 360.0 * units.deg)
    c0
        .add_rod_journal(rj0)
        .add_rod_journal(rj1)
        .add_rod_journal(rj2)
        .add_rod_journal(rj3)
        .add_rod_journal(rj4)
        .add_rod_journal(rj5)

    piston_parameters piston_params(
        mass: (50) * units.g, // 414 - piston mass, 152 - pin weight
        compression_height: compression_height,
        wrist_pin_position: 0.0,
        displacement: 0.0
    )

    connecting_rod_parameters cr_params(
        mass: rod_mass,
        moment_of_inertia: rod_moment_of_inertia(
            mass: rod_mass,
            length: rod_length
        ),
        center_of_mass: 0.0,
        length: rod_length
    )
    intake intake(
        plenum_volume: 1 * units.L,
        plenum_cross_section_area: 20 * units.cm2,
        intake_flow_rate: k_carb(3000),
        runner_flow_rate: k_carb(1600),
        runner_length: 16 * units.inch,
        idle_flow_rate: k_carb(0.01),
        idle_throttle_plate_position: 0.9995,
        velocity_decay: 0.5
    )
    exhaust_system_parameters es_params(
        outlet_flow_rate: k_carb(2000.0),
        primary_tube_length: 20.0 * units.inch,
        primary_flow_rate: k_carb(800.0),
        velocity_decay: 0.5
    )
    exhaust_system exhaust0(
        es_params,
        audio_volume: 1.0 * 0.004,
        length: 132 * units.inch,
        impulse_response: ir_lib.mild_exhaust_0_reverb
    )

    cylinder_bank_parameters bank_params(
        bore: bore,
        deck_height: 208.5 * units.mm
    )

    label spacing(0.0)
    cylinder_bank b0(bank_params, angle: 0 * units.deg)
    b0
        .add_cylinder(
            piston: piston(piston_params, blowby: k_28inH2O(0)),
            connecting_rod: connecting_rod(cr_params),
            rod_journal: rj0,
            intake: intake,
            exhaust_system: exhaust0,
            ignition_wire: wires.wire0,
            sound_attenuation: 0.5868734241863719,
            primary_length: 0 * spacing * 0.5 * units.cm
        )
        .add_cylinder(
            piston: piston(piston_params, blowby: k_28inH2O(0)),
            connecting_rod: connecting_rod(cr_params),
            rod_journal: rj1,
            intake: intake,
            exhaust_system: exhaust0,
            ignition_wire: wires.wire1,
            sound_attenuation: 0.788357003805589,
            primary_length: 1 * spacing * 0.5 * units.cm
        )
        .add_cylinder(
            piston: piston(piston_params, blowby: k_28inH2O(0)),
            connecting_rod: connecting_rod(cr_params),
            rod_journal: rj2,
            intake: intake,
            exhaust_system: exhaust0,
            ignition_wire: wires.wire2,
            sound_attenuation: 0.8617434783599845,
            primary_length: 2 * spacing * 0.5 * units.cm
        )
        .add_cylinder(
            piston: piston(piston_params, blowby: k_28inH2O(0)),
            connecting_rod: connecting_rod(cr_params),
            rod_journal: rj3,
            intake: intake,
            exhaust_system: exhaust0,
            ignition_wire: wires.wire3,
            sound_attenuation: 0.9768918961501529,
            primary_length: 3 * spacing * 0.5 * units.cm
        )
        .add_cylinder(
            piston: piston(piston_params, blowby: k_28inH2O(0)),
            connecting_rod: connecting_rod(cr_params),
            rod_journal: rj4,
            intake: intake,
            exhaust_system: exhaust0,
            ignition_wire: wires.wire4,
            sound_attenuation: 0.6831130386757958,
            primary_length: 4 * spacing * 0.5 * units.cm
        )
        .add_cylinder(
            piston: piston(piston_params, blowby: k_28inH2O(0)),
            connecting_rod: connecting_rod(cr_params),
            rod_journal: rj5,
            intake: intake,
            exhaust_system: exhaust0,
            ignition_wire: wires.wire5,
            sound_attenuation: 0.8691341430222019,
            primary_length: 5 * spacing * 0.5 * units.cm
        )
        .set_cylinder_head(
            M4_head(
                intake_camshaft: camshaft.intake_cam_0,
                exhaust_camshaft: camshaft.exhaust_cam_0,
                flip_display: false,
                flow_attenuation: 1.0)
        )

    engine
        .add_cylinder_bank(b0)

    engine.add_crankshaft(c0)

    harmonic_cam_lobe intake_lobe(
        duration_at_50_thou: 234 * units.deg,
        gamma: 1.1,
        lift: 571 * units.thou,
        steps: 512
    )

    harmonic_cam_lobe exhaust_lobe(
        duration_at_50_thou: 235 * units.deg,
        gamma: 1.1,
        lift: 571 * units.thou,
        steps: 512
    )

    M4_camshaft camshaft(
        lobe_profile: "N/A",

        intake_lobe_profile: intake_lobe,
        exhaust_lobe_profile: exhaust_lobe,
        intake_lobe_center: 90 * units.deg,
        exhaust_lobe_center: 112 * units.deg
    )

    function timing_curve(1000 * units.rpm)
    timing_curve
        .add_sample(0 * units.rpm, 18 * units.deg)
        .add_sample(1000 * units.rpm, 40 * units.deg)
        .add_sample(2000 * units.rpm, 40 * units.deg)
        .add_sample(3000 * units.rpm, 40 * units.deg)
        .add_sample(4000 * units.rpm, 40 * units.deg)
        .add_sample(5000 * units.rpm, 40 * units.deg)
        .add_sample(6000 * units.rpm, 40 * units.deg)
        .add_sample(7000 * units.rpm, 40 * units.deg)
        .add_sample(8000 * units.rpm, 40 * units.deg)
        .add_sample(9000 * units.rpm, 40 * units.deg)

    ignition_module ignition_module(
        timing_curve: timing_curve,
        rev_limit: 15000 * units.rpm,
        limiter_duration: 0.05)

    ignition_module
            .connect_wire(wires.wire0, 0.0 * units.deg)
            .connect_wire(wires.wire4, 120.0 * units.deg)
            .connect_wire(wires.wire2, 240.0 * units.deg)
            .connect_wire(wires.wire5, 360.0 * units.deg)
            .connect_wire(wires.wire1, 480.0 * units.deg)
            .connect_wire(wires.wire3, 600.0 * units.deg)

    engine.add_ignition_module(ignition_module)
}

private node M4_vehicle {
    alias output __out:
        vehicle(
            mass: 1725 * units.kg,
            drag_coefficient: 0.34,
            cross_sectional_area: (72 * units.inch) * (36 * units.inch),
            diff_ratio: 3.15,
            tire_radius: 19 * units.inch,
            rolling_resistance: 200 * units.N
        );
}

private node M4_transmission {
    alias output __out:
        transmission(
            max_clutch_torque: 1000 * units.lb_ft
        )
    .add_gear(5)
    .add_gear(3.2)
    .add_gear(2.14)
    .add_gear(1.72)
    .add_gear(1.31)
    .add_gear(1)
    .add_gear(0.82)
    .add_gear(0.64);
}

public node main {
    run(
        engine: M4_engine(),
        vehicle: M4_vehicle(),
        transmission: M4_transmission()
    )
}

main()
