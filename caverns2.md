import sys
import time

# Function to calculate the Euclidean distance between two points
def euclidean_distance(coord1, coord2):
    return ((coord1[0] - coord2[0])**2 + (coord1[1] - coord2[1])**2)**0.5

# Revised function to parse the .cav file and extract the connectivity matrix correctly
def revised_parse_new_cav_file(filepath):
    with open(filepath, 'r') as file:
        data = file.read().replace('\n', '').split(',')
        data = [int(i) for i in data]
        num_caverns = data[0]
        coordinates = [(data[i], data[i + 1]) for i in range(1, num_caverns * 2 + 1, 2)]
        matrix_start = num_caverns * 2 + 1
        connectivity = [data[matrix_start + i:matrix_start + i + num_caverns] for i in range(0, num_caverns**2, num_caverns)]
        return num_caverns, coordinates, connectivity

# Function to find the shortest path using Dijkstra's algorithm without PriorityQueue
def find_shortest_path_and_length(num_caverns, coordinates, connectivity):
    distances = [float('inf')] * num_caverns
    distances[0] = 0
    previous = [None] * num_caverns
    visited = [False] * num_caverns
    while True:
        shortest_distance = float('inf')
        shortest_index = None
        for i in range(num_caverns):
            if distances[i] < shortest_distance and not visited[i]:
                shortest_distance = distances[i]
                shortest_index = i
        if shortest_index is None:
            break
        for i in range(num_caverns):
            if connectivity[shortest_index][i] and not visited[i]:
                new_distance = distances[shortest_index] + euclidean_distance(coordinates[shortest_index], coordinates[i])
                if new_distance < distances[i]:
                    distances[i] = new_distance
                    previous[i] = shortest_index
        visited[shortest_index] = True
        if shortest_index == num_caverns - 1:
            break
    path = []
    current_cavern = num_caverns - 1
    total_length = 0.0
    while current_cavern is not None:
        path.insert(0, current_cavern)
        if previous[current_cavern] is not None:
            total_length += euclidean_distance(coordinates[previous[current_cavern]], coordinates[current_cavern])
        current_cavern = previous[current_cavern]
    if path[0] == 0:
        return path, total_length
    else:
        return None, 0

# Function to write the solution to a .csn file
def write_solution_csn_file(filepath, route):
    with open(filepath, 'w') as file:
        if route is not None:
            # Increment each index by 1 for one-based indexing
            file.write(" ".join(map(lambda x: str(x + 1), route[0])) + "\n")
            file.write(f"Length: {route[1]:.2f}")
        else:
            file.write("0")

def main():
    if len(sys.argv) != 2:
        print("Please use 2 files (python file and cav file)")
        sys.exit(1)
    
    filepath = sys.argv[1]
    input_file = filepath + ".cav"
    csn_output = filepath + ".csn"

    num_caverns, coordinates, connectivity = revised_parse_new_cav_file(input_file)
    start_time = time.time()
    route = find_shortest_path_and_length(num_caverns, coordinates, connectivity)
    end_time = time.time()

    write_solution_csn_file(csn_output, route)
    print(f"Path: {' '.join(map(lambda x: str(x + 1), route[0]))}")
    print(f"Length: {route[1]:.2f}")
    print(f"Time taken: {end_time - start_time:.2f} seconds")

if __name__ == "__main__":
    main()
