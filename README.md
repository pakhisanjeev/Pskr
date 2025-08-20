#include <iostream>
#include <vector>
#include <queue>
#include <string>
#include <algorithm>
#include <climits>

using namespace std;

// Class representing a Shop with name, location (id), and products it offers
class Shop {
public:
    string name;              // Shop name
    int location;             // Location id (used in graph)
    vector<string> products;  // List of products
    vector<double> prices;    // Corresponding prices for products

    Shop(string n, int loc) : name(n), location(loc) {}

    // Add product and its price to the shop's inventory
    void addProduct(string product, double price) {
        products.push_back(product);
        prices.push_back(price);
    }

    // Find product index using LCS (Longest Common Subsequence) to match product names fuzzily
    int findProductIndex(string product) {
        int bestIndex = -1;
        int bestLCSLength = -1;
        for (int i = 0; i < (int)products.size(); i++) {
            int lcsLen = calculateLCS(products[i], product);
            if (lcsLen > bestLCSLength) {
                bestLCSLength = lcsLen;
                bestIndex = i;
            }
        }
        return bestIndex;
    }

private:
    // Utility function to calculate LCS length between two strings for fuzzy matching
    int calculateLCS(const string &a, const string &b) {
        int m = (int)a.size(), n = (int)b.size();
        vector<vector<int>> dp(m + 1, vector<int>(n + 1, 0));
        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                if (a[i - 1] == b[j - 1])
                    dp[i][j] = dp[i - 1][j - 1] + 1;
                else
                    dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]);
            }
        }
        return dp[m][n];
    }
};

// Data structure to hold shop and relevant info for sorting/display
struct ShopResult {
    int shopIndex;    // index of shop in main shops vector
    double distance;  // distance from user location to shop
    double price;     // price of product in that shop

    ShopResult(int si, double dist, double pr) : shopIndex(si), distance(dist), price(pr) {}
};

// Function to perform Dijkstra's algorithm to find shortest distances from source to all nodes
vector<double> dijkstra(int source, const vector<vector<pair<int, double>>> &graph) {
    int n = (int)graph.size();
    vector<double> dist(n, INT_MAX); // Distance initialized to infinity
    dist[source] = 0;                // Distance to source is zero
    priority_queue<pair<double, int>, vector<pair<double, int>>, greater<>> pq;
    pq.push({0, source});            // Push source with distance 0

    while (!pq.empty()) {
        auto [curDist, u] = pq.top(); pq.pop();
        if (curDist > dist[u]) continue; // Skip if already found a shorter path
        for (auto [v, weight] : graph[u]) {
            if (dist[u] + weight < dist[v]) {
                dist[v] = dist[u] + weight;  // Update distance if shorter path found
                pq.push({dist[v], v});       // Push updated distance for next exploration
            }
        }
    }
    return dist; // Return computed shortest distances from source
}

// Merge function used in merge sort for sorting shops by distance then price
void merge(vector<ShopResult> &arr, int left, int mid, int right) {
    int n1 = mid - left + 1;
    int n2 = right - mid;
    vector<ShopResult> L(arr.begin() + left, arr.begin() + mid + 1);
    vector<ShopResult> R(arr.begin() + mid + 1, arr.begin() + right + 1);

    int i = 0, j = 0, k = left;
    // Merge sorted parts by comparing distance and then price
    while (i < n1 && j < n2) {
        if (L[i].distance < R[j].distance || 
           (L[i].distance == R[j].distance && L[i].price < R[j].price)) {
            arr[k++] = L[i++];
        } else {
            arr[k++] = R[j++];
        }
    }
    while (i < n1) arr[k++] = L[i++];
    while (j < n2) arr[k++] = R[j++];
}

// Merge sort function to sort shops by distance and price
void mergeSort(vector<ShopResult> &arr, int left, int right) {
    if (left >= right) return;
    int mid = (left + right) / 2;
    mergeSort(arr, left, mid);
    mergeSort(arr, mid + 1, right);
    merge(arr, left, mid, right);
}

int main() {
    // Sample graph initialization: Adjacency list representation
    // Each vector index represents location, pair stores adjacent location and distance
    vector<vector<pair<int, double>>> graph = {
        {{1, 2.0}, {2, 4.0}},     // location 0 connected to 1 (2.0), 2 (4.0)
        {{0, 2.0}, {2, 1.0}, {3, 7.0}},
        {{0, 4.0}, {1, 1.0}, {3, 3.0}},
        {{1, 7.0}, {2, 3.0}}
    };

    // Create shops with products
    vector<Shop> shops;
    shops.emplace_back("ShopA", 1);
    shops.back().addProduct("Milk", 25.0);
    shops.back().addProduct("Bread", 30.0);

    shops.emplace_back("ShopB", 2);
    shops.back().addProduct("Milk", 20.0);
    shops.back().addProduct("Butter", 45.0);

    shops.emplace_back("ShopC", 3);
    shops.back().addProduct("Bread", 28.0);
    shops.back().addProduct("Milk", 27.0);

    int userLocation = 0;            // User's current location (node 0)
    string requiredProduct = "Milk"; // Product user searches for

    vector<double> distances = dijkstra(userLocation, graph); // Find shortest distances

    vector<ShopResult> results; // To hold shops that have the product

    // Search shops for product (using LCS matching), collect results with distances and price
    for (int i = 0; i < (int)shops.size(); i++) {
        int productIndex = shops[i].findProductIndex(requiredProduct);
        if (productIndex != -1) { // If product found in shop
            double dist = distances[shops[i].location]; // Distance from user
            double price = shops[i].prices[productIndex]; // Price of product in shop
            results.emplace_back(i, dist, price);
        }
    }

    // Sort results by distance first, then price
    mergeSort(results, 0, (int)results.size() - 1);

    // Display sorted results to user
    cout << "Nearest shops offering " << requiredProduct << " sorted by distance & price:\n";
    for (auto &res : results) {
        cout << shops[res.shopIndex].name << " at distance " << res.distance
             << " with price " << res.price << "\n";
    }

    return 0;
}
