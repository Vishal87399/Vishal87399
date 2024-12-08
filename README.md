import random
import networkx as nx
from geopy.distance import geodesic
import heapq

class RideSharingPlatform:
    def __init__(self):
        self.vehicles = []
        self.requests = []
        self.graph = nx.Graph()

    def add_vehicle(self, vehicle_id, location):
        """Add a vehicle to the platform."""
        self.vehicles.append({'id': vehicle_id, 'location': location, 'available': True})
        print(f"Vehicle {vehicle_id} added at location {location}.")

    def add_request(self, user_id, start_location, end_location):
        """Add a ride request."""
        self.requests.append({'user_id': user_id, 'start': start_location, 'end': end_location})
        print(f"Request from User {user_id} added: {start_location} -> {end_location}.")

    def build_route_graph(self, locations):
        """Build a simple graph to simulate routes between locations."""
        self.graph.clear()
        for loc1 in locations:
            for loc2 in locations:
                if loc1 != loc2:
                    # Distance as weight (using geodesic distance)
                    distance = geodesic(loc1, loc2).km
                    self.graph.add_edge(loc1, loc2, weight=distance)

    def find_closest_vehicle(self, start_location):
        """Find the closest available vehicle using distance."""
        min_distance = float('inf')
        closest_vehicle = None
        for vehicle in self.vehicles:
            if vehicle['available']:
                distance = geodesic(vehicle['location'], start_location).km
                if distance < min_distance:
                    min_distance = distance
                    closest_vehicle = vehicle
        return closest_vehicle, min_distance

    def assign_vehicle(self):
        """Assign the closest vehicle to a ride request."""
        for request in self.requests:
            vehicle, distance = self.find_closest_vehicle(request['start'])
            if vehicle:
                vehicle['available'] = False
                print(f"Assigned Vehicle {vehicle['id']} to User {request['user_id']}. Distance: {distance:.2f} km")
                self.calculate_optimal_route(request['start'], request['end'])
            else:
                print(f"No available vehicle for User {request['user_id']}.")

    def calculate_optimal_route(self, start_location, end_location):
        """Calculate the shortest path using Dijkstra's algorithm."""
        try:
            route = nx.shortest_path(self.graph, source=start_location, target=end_location, weight='weight')
            distance = nx.shortest_path_length(self.graph, source=start_location, target=end_location, weight='weight')
            print(f"Optimal route from {start_location} to {end_location}: {route}. Distance: {distance:.2f} km")
        except nx.NetworkXNoPath:
            print(f"No path found between {start_location} and {end_location}.")

# Example usage
platform = RideSharingPlatform()

# Add vehicles and requests
platform.add_vehicle("V1", (37.7749, -122.4194))  # San Francisco
platform.add_vehicle("V2", (34.0522, -118.2437))  # Los Angeles
platform.add_request("U1", (37.7749, -122.4194), (34.0522, -118.2437))

# Build graph with mock locations
locations = [
    (37.7749, -122.4194),  # San Francisco
    (34.0522, -118.2437),  # Los Angeles
    (36.1699, -115.1398)   # Las Vegas
]
platform.build_route_graph(locations)

# Assign vehicles to requests and calculate routes
platform.assign_vehicle()
