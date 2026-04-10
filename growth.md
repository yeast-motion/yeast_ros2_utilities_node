# yeast_ros2_utilities_node - Growth Opportunities

## Improvement Opportunities

### 1. Pass all conversion function parameters by `const&`
All `to_msg()` and `from_msg()` functions in `src/yeast_msg_helper.cpp` should take parameters by const reference to avoid unnecessary copies of potentially deep object trees.

### 2. Remove hardcoded paths from CMakeLists.txt
Replace `/mnt/working/third_party_libs` (line 14) and `/usr/local/include/wpimath` (line 81) with `find_package` calls or configurable CMake variables.

### 3. Link to yeastcpp via CMake target instead of direct .a path
Replace `${THIRD_PARTY_LIB_FOLDER}/yeastcpp/build/libyeastcpp.a` (line 98) with a proper `find_package(yeastcpp)` or `add_subdirectory()` approach.

### 4. Add `Trajectory` conversion functions
No `to_msg`/`from_msg` exists for `Trajectory` since there is no corresponding ROS2 message. Adding this would enable trajectory passing over ROS2 topics.

### 5. Auto-generate array/vector conversion helpers
Currently there is no helper for `std::vector<SwerveModuleCommand>` or `std::vector<SwerveModuleStatus>`. Callers must loop manually. The Jinja template could generate vector overloads.

## Architectural Enhancements

### 6. Ensure generated code is always regenerated at build time
The CMake custom command at lines 49-64 generates the code, but the committed generated files may be stale. Consider adding the generated files to `.gitignore` and always generating fresh at build time.

### 7. Add bidirectional round-trip validation
The `to_msg(from_msg(msg))` should produce an identical message. This is currently not validated and is known to fail for `OdometrySample` (missing validity fields) and `SwerveModuleCommand` (missing `accel`).

### 8. Remove unused `ck_ros2_base_msgs_node` dependency
If the utilities node does not actually use `ck_ros2_base_msgs_node`, remove it from `package.xml:14` and `CMakeLists.txt:28` to reduce build dependencies.

## Testing Gaps

### 9. No unit tests
Critical areas needing tests:
- Round-trip conversion: `from_msg(to_msg(yeast_type)) == yeast_type` for every type
- `to_msg(from_msg(msg)) == msg` for every type  
- Conversion of edge values (NaN, infinity, very large/small floats)
- Verify the generated code matches what the Jinja templates would produce from current schemas

### 10. No integration tests with actual ROS2 message passing
No tests verify that the messages survive serialization/deserialization through ROS2 DDS.
