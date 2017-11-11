# gear [![Build Status](https://travis-ci.org/OTL/gear.svg?branch=master)](https://travis-ci.org/OTL/gear) [![crates.io](https://img.shields.io/crates/v/gear.svg)](https://crates.io/crates/gear)

Collision Avoidance Path Planning for robotics in Rust-lang


[![Video](https://j.gifs.com/kZZyJK.gif)](http://www.youtube.com/watch?v=jEu3EfpVAI8)

## Code example

### [minimum code example](examples/minimum.rs)

```rust
extern crate gear;
extern crate nalgebra as na;

fn main() {
    // Create path planner with loading urdf file and set end link name
    let planner = gear::JointPathPlannerBuilder::try_from_urdf_file("sample.urdf", "l_wrist2")
        .expect("failed to create planner from urdf file")
        .collision_check_margin(0.01)
        .finalize();
    // Create inverse kinematics solver
    let solver = gear::JacobianIKSolverBuilder::<f64>::new()
        .num_max_try(1000)
        .allowable_target_distance(0.01)
        .move_epsilon(0.0001)
        .finalize();
    let solver = gear::RandomInitializeIKSolver::new(solver, 100);
    // Create path planner with IK solver
    let mut planner = gear::JointPathPlannerWithIK::new(planner, solver);

    // Create obstacles
    let obstacles =
        gear::create_compound_from_urdf("obstacles.urdf").expect("obstacle file not found");

    // Set IK target transformation
    let mut ik_target_pose = na::Isometry3::from_parts(
        na::Translation3::new(0.40, 0.20, 0.3),
        na::UnitQuaternion::from_euler_angles(0.0, -0.1, 0.0),
    );
    // Plan the path, path is the vector of joint angles for root to "l_wrist2"
    let plan1 = planner.plan_with_ik(&ik_target_pose, &obstacles).unwrap();
    println!("plan1 = {:?}", plan1);
    ik_target_pose.translation.vector[2] += 0.50;
    // plan the path from previous result
    let plan2 = planner.plan_with_ik(&ik_target_pose, &obstacles).unwrap();
    println!("plan2 = {:?}", plan2);
}
```

## Run example with GUI

### How to run

```bash
cargo run --release --example reach
```

### GUI control

* Up/Down/Left/Right/`f`/`b` to move IK target
* type `g` to move the end of the arm to the target

* type `i` to just solve inverse kinematics for the target
* type `r` to set random pose
* type `c` to check collision

### Use your robot

The example can handle any urdf files (sample.urdf is used as default).
It requires the name of the target end link name.

```bash
cargo run --release --example reach YOUR_URDF_FILE_PATH END_LINK_NAME
```

For example, you can use PR2.

```bash
cargo run --release --example reach $(rospack find pr2_description)/robots/pr2.urdf.xacro l_gripper_palm_link
```
