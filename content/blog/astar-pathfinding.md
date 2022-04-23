+++
title = "Astar Pathfinding"
date = 2022-04-22
description = "One of the most common ways to find the cheapest path from A to B"
in_search_index = true
[taxonomies]
tags = ["rust", "gamedev"]
+++
One important aspect of A\* is f = g + h. The f, g, and h variables are in our Node class and get calculated every time we create a new node. Quickly I’ll go over what these variables mean.
F is the total cost of the node.
G is the distance between the current node and the start node.
H is the heuristic — estimated distance from the current node to the end node.
```rust
#[derive(Debug, PartialEq, Eq, Clone)]
struct PathNode<'a> {
    cell: &'a Cell,
    g_cost: i32,
    h_cost: i32,
    f_cost: i32,
    came_from: Box<Option<PathNode<'a>>>,
}
```
If you’re a game developer, you might have always wanted to implement A\* as character pathfinding.  I wanted to write this as an easy introduction with a clear source code example to A\* pathfinding so anyone can easily pick it up and use it in their game.
I’ve pasted the steps for A* from Patrick Lester’s article that you can check out here. The same website is also listed below in resources. This is an insanely good explanation, and is why I decided to go with it rather than writing it again.
1. Add the starting square (or node) to the open list.
2. Repeat the following:
	* Look for the lowest F cost square on the open list. We refer to this as the current square.
	* Switch it to the closed list.
	* For each of the 8 squares adjacent to this current square …
		* If it is not walkable or if it is on the closed list, ignore it. Otherwise do the following.
		* If it isn’t on the open list, add it to the open list. Make the current square the parent of this square. Record the F, G, and H costs of the square.
		* If it is on the open list already, check to see if this path to that square is better, using G cost as the measure. A lower G cost means that this is a better path. If so, change the parent of the square to the current square, and recalculate the G and F scores of the square. If you are keeping your open list sorted by F score, you may need to resort the list to account for the change.
	* Stop when you:
		* Add the target square to the closed list, in which case the path has been found, or
		* Fail to find the target square, and the open list is empty. In this case, there is no path.
3. Save the path. Working backwards from the target square, go from each square to its parent square until you reach the starting square. That is your path.

```rust
const DIRECTIONAL_ARRAY_X: [i32; 8] = [-1, 0, 1, -1, 1, -1, 0, 1];
const DIRECTIONAL_ARRAY_Y: [i32; 8] = [-1, -1, -1, 0, 0, 1, 1, 1];
const DIST_BETWEEN: i32 = 10;
fn heuristic(a: &Cell, b: &Cell) -> i32 {
    let diff_x: i32 = (a.x as i32 - b.x as i32).abs();
    let diff_y: i32 = (a.y as i32 - b.y as i32).abs();
    diff_x + diff_y
}
pub fn find_path(game: &GameData, start_cell: &Cell, end_cell: &Cell) -> Option<Vec<Cell>> {
    let start_node: PathNode = PathNode {
        cell: start_cell,
        g_cost: 0,
        h_cost: 0,
        f_cost: 0,
        came_from: Box::new(None),
    };
    let end_node: PathNode = PathNode {
        cell: end_cell,
        g_cost: 0,
        h_cost: 0,
        f_cost: 0,
        came_from: Box::new(None),
    };

    let mut open_set: Vec<PathNode> = Vec::new();
    let mut closed_set: Vec<PathNode> = Vec::new();

    open_set.push(start_node);

    while open_set.len() > 0 {
        let mut lowest_index = 0;
        for i in 0..open_set.len() {
            if open_set[i].f_cost < open_set[lowest_index].f_cost {
                lowest_index = i;
            }
        }
        let current: PathNode = open_set.remove(lowest_index);

        if current.cell == end_node.cell {
            let mut path: Vec<Cell> = Vec::new();
            let mut path_node: &PathNode = &current;
            path.push(current.cell.clone());
            while let Some(previous) = &*path_node.came_from {
                path.push(previous.cell.clone());
                path_node = previous;
            }
            return Some(path);
        }

        // Find neighbors
        for i in 0..8 {
            let x = current.cell.x as i32 + DIRECTIONAL_ARRAY_X[i];
            let y = current.cell.y as i32 + DIRECTIONAL_ARRAY_Y[i];

            if x < 0
               || x >= game.board_size_x
               || y < 0
               || y >= game.board_size_y
               || closed_set.iter().any(|node| node.cell == &game.board[y as usize][x as usize])
               || game.board[y as usize][x as usize].solid
               {
                continue;
            }

            let tentative_g_score = if DIRECTIONAL_ARRAY_X[i] == 0 || DIRECTIONAL_ARRAY_Y[i] == 0 {
                current.g_cost + DIST_BETWEEN // Orthogonal movement
            } else {
                current.g_cost + (DIST_BETWEEN as f32 * DIST_BETWEEN as f32 * 2.0).sqrt() as i32 // Diagonal movement
            };

            if let Some(neighbor) = open_set.iter().find(|&node| node.cell == &game.board[y as usize][x as usize]) {
                if neighbor.g_cost <= tentative_g_score {
                    continue;
                }
            } else {
                let h_score = heuristic(&game.board[y as usize][x as usize], &end_node.cell);
                let neighbor: PathNode = PathNode {
                    cell: &game.board[y as usize][x as usize],
                    g_cost: tentative_g_score,
                    h_cost: h_score,
                    f_cost: tentative_g_score + h_score,
                    came_from: Box::new(Some(current.clone())),
                };
                open_set.push(neighbor);
            }
        }
        closed_set.push(current);
    }
    return None;
}
```
