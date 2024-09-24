---
layout: default
title: Flow Recorder
parent: New Implementation
nav_order: 5
---

# New Implementation: Flow Recorder

The Flow Recorder is a new component introduced in the latest implementation to enhance the analysis, debugging, and visualization capabilities of the flow algorithms. It provides a way to capture the step-by-step progress of flow computations and generate visual representations of the process.

## Structure

The Flow Recorder is implemented as a struct with methods for recording steps and generating visualizations:

```rust
pub struct FlowRecorder {
    steps: Vec<FlowStep>,
    all_nodes: HashSet<Address>,
}

struct FlowStep {
    current_flow: U256,
    path: Vec<Address>,
    path_flow: U256,
    graph_state: FlowGraph,
}
```

## Key Components

### 1. Step Recording

The `record_step` method captures the state of the flow computation at each iteration:

```rust
pub fn record_step(&mut self, current_flow: U256, path: Vec<Address>, path_flow: U256, graph_state: FlowGraph) {
    self.all_nodes.extend(path.iter().cloned());
    self.steps.push(FlowStep {
        current_flow,
        path,
        path_flow,
        graph_state,
    });
}
```

### 2. Visualization Generation

The `generate_visualization` method creates a GIF animation of the flow computation process:

```rust
pub fn generate_visualization(&self, output_path: &str, source: Address, sink: Address) -> Result<(), Box<dyn std::error::Error>> {
    let width: u32 = 1600;
    let height: u32 = 900;
    let mut encoder = Encoder::new(std::fs::File::create(output_path)?, width as u16, height as u16, &[])?;
    encoder.set_repeat(Repeat::Infinite)?;

    let node_positions = self.calculate_node_positions(source, sink, width, height);
    let mut previous_paths: Vec<Vec<Address>> = Vec::new();

    for (i, step) in self.steps.iter().enumerate() {
        let mut image = RgbaImage::new(width, height);

        // Clear the image
        for pixel in image.pixels_mut() {
            *pixel = Rgba([255, 255, 255, 255]);
        }

        // Draw previous paths in gray
        for path in &previous_paths {
            self.draw_path(&mut image, path, &Rgba([200, 200, 200, 255]), &node_positions);
        }

        // Draw current path in red
        self.draw_path(&mut image, &step.path, &Rgba([255, 0, 0, 255]), &node_positions);

        // Draw source and sink nodes
        self.draw_node(&mut image, &source, &node_positions, Rgba([0, 255, 0, 255]));
        self.draw_node(&mut image, &sink, &node_positions, Rgba([0, 0, 255, 255]));

        // Add text information
        self.draw_text(&mut image, 10, 10, &format!("Step: {}", i + 1));
        self.draw_text(&mut image, 10, 30, &format!("Current Total Flow: {} tokens", step.current_flow));
        self.draw_text(&mut image, 10, 50, &format!("Path Flow: {} tokens", step.path_flow));
        self.draw_text(&mut image, 10, 70, "Green: Source, Blue: Sink, Red: Active path");
        self.draw_text(&mut image, 10, 90, "Gray: Previous paths");

        // Convert the image to a GIF frame and write it
        let mut frame = Frame::from_rgba(width as u16, height as u16, &mut image.into_raw());
        frame.delay = 200; // 2 seconds delay
        encoder.write_frame(&frame)?;

        // Add current path to previous paths for next iteration
        previous_paths.push(step.path.clone());
    }

    Ok(())
}
```

## Visualization Techniques

The Flow Recorder uses several techniques to create an informative and visually appealing representation of the flow computation process:

1. **Node Positioning**: Nodes are positioned using a level-based layout algorithm, ensuring that the source and sink are at opposite ends and intermediate nodes are distributed evenly.

```rust
fn calculate_node_positions(
    &self,
    source: Address,
    sink: Address,
    width: u32,
    height: u32,
) -> HashMap<Address, (u32, u32)> {
    let mut node_positions = HashMap::new();
    let margin = 100u32;

    // Build node levels mapping
    let mut node_levels = HashMap::new();

    // Process all paths to assign levels to nodes
    for step in &self.steps {
        for (index, &node) in step.path.iter().enumerate() {
            // Assign the earliest level
            node_levels
                .entry(node)
                .and_modify(|e| if index < *e { *e = index })
                .or_insert(index);
        }
    }

    // Adjust levels so that source is at level 0
    node_levels.insert(source, 0);

    // Adjust levels so that sink is at the highest level + 1
    let max_level = node_levels.values().max().cloned().unwrap_or(0);
    let sink_level = max_level + 1;
    node_levels.insert(sink, sink_level);

    // Update max_level after inserting sink
    let max_level = sink_level;

    // Build levels to nodes mapping
    let mut levels_to_nodes: HashMap<usize, Vec<Address>> = HashMap::new();
    for (&node, &level) in &node_levels {
        levels_to_nodes.entry(level).or_default().push(node);
    }

    // Assign positions based on levels
    let level_width = (width - 2 * margin) / (max_level as u32);

    for level in 0..=max_level {
        let x = margin + (level as u32 * level_width);
        if let Some(nodes_at_level) = levels_to_nodes.get(&level) {
            let num_nodes = nodes_at_level.len() as u32;
            for (i, &node) in nodes_at_level.iter().enumerate() {
                // Spread nodes vertically
                let y = margin + ((height - 2 * margin) / (num_nodes + 1)) * (i as u32 + 1);
                node_positions.insert(node, (x, y));
            }
        }
    }

    node_positions
}
```

2. **Path Drawing**: Paths are drawn as directed arrows between nodes, with different colors representing the current path and previous paths.

```rust
fn draw_path(
    &self,
    image: &mut RgbaImage,
    path: &[Address],
    color: &Rgba<u8>,
    node_positions: &HashMap<Address, (u32, u32)>,
) {
    for window in path.windows(2) {
        let from = window[0];
        let to = window[1];
        if let (Some(&(x1, y1)), Some(&(x2, y2))) = (node_positions.get(&from), node_positions.get(&to)) {
            self.draw_arrow(image, (x1, y1), (x2, y2), *color);

            // Draw nodes of the path
            self.draw_node(image, &to, node_positions, *color);
        }
    }
}

fn draw_arrow(&self, image: &mut RgbaImage, from: (u32, u32), to: (u32, u32), color: Rgba<u8>) {
    let dx = (to.0 as f32 - from.0 as f32);
    let dy = (to.1 as f32 - from.1 as f32);
    let length = (dx * dx + dy * dy).sqrt();
    let unit_x = dx / length;
    let unit_y = dy / length;

    // Calculate midpoint
    let mid_x = (from.0 as f32 + to.0 as f32) / 2.0;
    let mid_y = (from.1 as f32 + to.1 as f32) / 2.0;

    let arrow_size: f32 = 15.0;
    let arrow_angle: f32 = 0.5;

    // Draw the main line of the edge
    draw_line_segment_mut(
        image,
        (from.0 as f32, from.1 as f32),
        (to.0 as f32, to.1 as f32),
        color,
    );

    // Calculate arrowhead position at the midpoint
    let tip_x = mid_x;
    let tip_y = mid_y;

    let left_x = tip_x - arrow_size * (unit_x * arrow_angle.cos() + unit_y * arrow_angle.sin());
    let left_y = tip_y - arrow_size * (-unit_x * arrow_angle.sin() + unit_y * arrow_angle.cos());
    let right_x = tip_x - arrow_size * (unit_x * arrow_angle.cos() - unit_y * arrow_angle.sin());
    let right_y = tip_y - arrow_size * (unit_x * arrow_angle.sin() + unit_y * arrow_angle.cos());

    let arrow_points = vec![
        Point::new(tip_x as i32, tip_y as i32),
        Point::new(left_x as i32, left_y as i32),
        Point::new(right_x as i32, right_y as i32),
    ];

    // Draw the arrowhead
    draw_polygon_mut(image, &arrow_points, color);
}
```

3. **Node Drawing**: Nodes are drawn as filled circles with labels.

```rust
fn draw_node(
    &self,
    image: &mut RgbaImage,
    node: &Address,
    node_positions: &HashMap<Address, (u32, u32)>,
    color: Rgba<u8>,
) {
    if let Some(&(x, y)) = node_positions.get(node) {
        draw_filled_circle_mut(image, (x as i32, y as i32), 20, color);

        let text_x = x.saturating_sub(15);
        let text_y = y.saturating_sub(7);

        self.draw_text(image, text_x, text_y, &node.short());
    }
}
```

4. **Text Annotations**: Each frame includes text annotations providing information about the current step, total flow, and path flow.

```rust
fn draw_text(&self, image: &mut RgbaImage, x: u32, y: u32, text: &str) {
    let font_data = include_bytes!("../assets/Roboto/Roboto-Black.ttf") as &[u8];
    let font = Font::try_from_bytes(font_data).unwrap();
    let scale = Scale::uniform(24.0);
    let color = Rgba([0, 0, 0, 255]);

    imageproc::drawing::draw_text_mut(image, color, x, y, scale, &font, text);
}
```

## Sample Visualization

Here's a description of what a typical frame in the generated GIF might look like:

1. **Network Layout**: Nodes are arranged in levels from left to right, with the source node (green) on the far left and the sink node (blue) on the far right.

2. **Current Path**: The path being augmented in the current step is highlighted in red, with arrows showing the direction of flow.

3. **Previous Paths**: Paths from previous steps are shown in gray, providing context for how the flow has evolved.

4. **Text Information**: At the top left of the frame, you'll see:
   - The current step number
   - The total flow achieved so far
   - The flow being pushed along the current path

5. **Color Legend**: At the bottom left, there's a color legend explaining what each color represents.

## Interpreting the Visualization

When viewing the generated GIF, you can observe the following:

1. **Flow Evolution**: Watch how the total flow increases with each step, and how the algorithm finds different paths through the network.

2. **Path Diversity**: Notice how the algorithm explores different paths through the network. This is particularly interesting when comparing different path search strategies.

3. **Bottlenecks**: Pay attention to edges that are frequently used (appearing in multiple paths). These might represent bottlenecks in the network.

4. **Algorithm Behavior**: Observe how the behavior changes as the algorithm progresses. For example, with the Capacity Scaling algorithm, you might notice it finding larger capacity paths first, then gradually using smaller capacity edges.

5. **Termination**: Watch for when the algorithm stops finding new paths, indicating it has reached the maximum flow or the requested flow.

## Usage Example

Here's an example of how to use the Flow Recorder in conjunction with a flow algorithm:

```rust
let mut recorder = FlowRecorder::new();
let (flow, transfers) = network_flow.compute_flow(
    algorithm,
    &from_address,
    &to_address,
    parsed_value_param,
    None,  // max_distance
    None,  // max_transfers
    path_search_algorithm,
    Some(&mut recorder)
);

// Generate visualization
recorder.generate_visualization("flow_visualization.gif", from_address, to_address)?;
```

## Conclusion

The Flow Recorder is a powerful addition to the new implementation, providing valuable insights into the flow computation process. Its ability to capture and visualize the step-by-step progression of flow algorithms enhances both the development process and the understanding of how these algorithms operate in complex networks.

The generated visualizations serve multiple purposes:

1. **Debugging**: They allow developers to visually inspect the algorithm's behavior, making it easier to identify issues or unexpected behaviors.

2. **Performance Analysis**: By observing how quickly the flow increases and how diverse the paths are, one can gain insights into the algorithm's efficiency.

3. **Education**: These visualizations are excellent tools for teaching and learning about network flow algorithms, as they make the abstract concepts more concrete and understandable.

4. **Algorithm Comparison**: By generating visualizations for different algorithms or parameters, it's easy to compare their behavior and effectiveness visually.

While the Flow Recorder does introduce some computational and memory overhead, especially for large graphs or long-running computations, the benefits in terms of debugging, analysis, and education make it a valuable tool in the network flow toolkit. Its integration into the new implementation demonstrates the focus on not just computational efficiency, but also on providing tools for better understanding and improving the algorithms.