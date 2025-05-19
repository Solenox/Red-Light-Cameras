# Red-Light-Cameras
// This program analyzes red light camera violation data, allowing users to explore trends through a simple menu. The program reads and processes camera records, violation counts, intersection locations, as well as the associated neighborhoods, and stores this data to be processed. From this it provides the user with a data overview, neighborhood analysis, a monthly violation chart, and a camera search feature.

#include <iostream>
#include <string>
#include <vector>
#include <iomanip>
#include <fstream>
#include <algorithm>

using namespace std;

// Class to store data for a single red ligh camera record
class CameraRecord {
private:
    string intersection;    // Intersection where the camera is located
    string address;         // Address of the camera location
    int cameraNum;          // Camera identifier number
    string date;            // Date of the violation
    int violations;         // Number of violations recorded
    string neighborhoods;   // Neighborhood where the camera is located

public:
    // Constructor to initialize a CameraRecord object
    CameraRecord(string inter, string addr, int cam, string d, int viol, string nbhd) {
        intersection = inter;
        address = addr;
        cameraNum = cam;
        date = d;
        violations = viol;
        neighborhoods = nbhd;
    }

    // Getter functions to access private attributes
    string getIntersection() const { return intersection ; }
    string getAddress() const { return address; }
    int getCameraNum() const { return cameraNum; }
    string getDate() const { return date; }
    int getViolations() const { return violations; }
    string getNeighborhoods() const { return neighborhoods; }
};

// Class to store data about a neighborhood's camera statistics
class Neighborhood {
private:
    string name;        // Name of the neighborhood
    int cameras;        // Number of cameras in the neighborhood
    int violations;     // Total violations in the neighborhood

public:
    // Constructor to intialize a Neighborhood object
    Neighborhood(string n, int c, int v) {
        name = n;
        cameras = c;
        violations = v;
    }

    // Getter functions used to access private attributes
    string getName() const { return name; }
    int getCameras() const { return cameras; }
    int getViolations() const { return violations; }

    // Used to sort neighborhoods by violations in descending order
    bool operator<(const Neighborhood& other) const { return violations > other.violations; }
};

// Converts a string to an integer by extracting digits at the beginning
int stringToInt(const string& str) {
    int num = 0;
    for (char c : str) {
        if (isdigit(c)) {
            num = num * 10 + (c - '0');
        } else {
            break;
        }
    }
    return num;
}

// Reads camera data from a file and stores it in a vector of CameraRecord objects
vector<CameraRecord> readFile(string filename) {
    vector<CameraRecord> records;
    ifstream fileIn(filename);

    // Checks if file opens successfully
    if (!fileIn.is_open()) {
        cout << "Error opening file" << endl;
        return records;
    }

    string line;
    // Reads each line of the file
    while (getline(fileIn, line)) {
        string intersection, address, date, neighborhood;
        int cameraNum, violations;
        size_t pos = 0;

        // Parses the line and extracts data
        pos = line.find(",");
        intersection = line.substr(0, pos);
        line.erase(0, pos + 1);

        pos = line.find(",");
        address = line.substr(0, pos);
        line.erase(0, pos + 1);

        pos = line.find(",");
        cameraNum = stringToInt(line.substr(0, pos));
        line.erase(0, pos + 1);

        pos = line.find(",");
        date = line.substr(0, pos);
        line.erase(0, pos + 1);

        pos = line.find(",");
        violations = stringToInt(line.substr(0, pos));
        line.erase(0, pos + 1);

        neighborhood = line; // The last field

        // Creates a CameraRecord object and adds it to the vector
        records.push_back(CameraRecord(intersection, address, cameraNum, date, violations, neighborhood));
    }
    fileIn.close();
    return records;
}

// Displays a general summary of camera violation data
void dataOverview(const vector<CameraRecord>& records) {
    vector<int> uniqueCameras;
    int totalViolations = 0;
    int maxViolations = 0;
    string maxIntersection, maxDate;

    // Loops through all records to calculate the summary data
    for (const auto& record : records) {
        // Checks if the camera is already counted
        bool found = false;
        for (int cam : uniqueCameras) {
            if (cam == record.getCameraNum()) {
                found = true;
                break;
            }
        }
        if (!found) uniqueCameras.push_back(record.getCameraNum());

        // Add to total violations
        totalViolations += record.getViolations();

        // Tracks the highest violation record
        if (record.getViolations() > maxViolations) {
            maxViolations = record.getViolations();
            maxIntersection = record.getIntersection();
            string date = record.getDate();
            string year = date.substr(0,4);
            string month = date.substr(5, 2);
            string day = date.substr(7, 2);
            int monthInt = stringToInt(month);
            int dayInt = stringToInt(day);
            maxDate = month + day + "-" + year;
        }    
    }

    // Displays the summary
    cout << "Read file with " << records.size() << " records." << endl;
    cout << "There are " << uniqueCameras.size() << " cameras." << endl;
    cout << "A total of " << totalViolations << " violations." << endl;
    cout << "The most violations in one day were " << maxViolations << " on " << maxDate << " at " << maxIntersection << endl;
}

// Displays the number of cameras and violations for each neighborhood
void resultsByNeighborhood(const vector<CameraRecord>& records) {
    vector<string> neighborhoodNames;
    vector<Neighborhood> neighborhoods;

    // Collects unique neighborhood names
    for (const auto& record : records) {
        bool found = false;
        for (const string& name : neighborhoodNames) {
            if (name == record.getNeighborhoods()) {
                found = true;
                break;
            }
        }
        if (!found) neighborhoodNames.push_back(record.getNeighborhoods());
    }

    // Calculates cameras and violations for each neighborhood
    for (const string& name : neighborhoodNames) {
        int cameras = 0;
        int violations = 0;
        vector<int> cameraIds;

        // Loops through records for each neighborhood
        for (const auto& record : records) {
            if (record.getNeighborhoods() == name) {
                violations += record.getViolations();
                bool found = false;
                for (int id : cameraIds) {
                    if (id == record.getCameraNum()) {
                        found = true;
                        break;
                    }
                }
                if (!found) {
                    cameraIds.push_back(record.getCameraNum());
                    cameras++;
                }
            }
        }
        neighborhoods.push_back(Neighborhood(name, cameras, violations));
    }
    sort(neighborhoods.begin(), neighborhoods.end());   // Sorts neighborhoods by violations

    // Displays the neighborhood data
    for(const auto& nbhd : neighborhoods) {
        cout << left << setw(25) << nbhd.getName() << right << setw(4) << nbhd.getCameras() << setw(7) << nbhd.getViolations() << endl;
    }
}

// Displays a chart of violations by month
void chartByMonth(const vector<CameraRecord>& records) {
    string months[] = {"January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November", "December"};
    int violations[12] = {0};

    // Counts violations per month
    for (const auto& record : records) {
        string date = record.getDate();
        int month = stringToInt(date.substr(5, 2)) - 1;
        violations[month] += record.getViolations();
    }

    // Displays the chart with stars representing the number of violations
    for (int i = 0; i < 12; i++) {
        int stars = violations[i] / 1000;
        cout << left << months[i] + string(15 - months[i].length(), ' ');
        for (int j = 0; j < stars; j++) {
            cout << "*";
        }
        cout << endl;
    }
}

// Converts a string to lowercase
string toLower(string str) {
    string result = str;
    for (char& c : result) {
        c = tolower(c);
    }
    return result;
}

// Searches for cameras based on user's input
void searchCameras(const vector<CameraRecord>& records) {
    string search;
    cout << "What should we search for? ";
    cin.ignore();
    getline(cin, search);
    search = toLower(search);

    vector<int> displayedCameras;
    bool foundAny = false;

    // Searches for cameras matching the search criteria
    for(const auto& record : records) {
        string intersection = toLower(record.getIntersection());
        string neighborhood = toLower(record.getNeighborhoods());
        bool alreadyDisplayed = false;

        for (int cam : displayedCameras) {
            if (cam == record.getCameraNum()) {
                alreadyDisplayed = true;
                break;
            }
        }

        // Displays camera if the search term matches either intersection or neighborhood
        if (!alreadyDisplayed && (intersection.find(search) != -1 || neighborhood.find(search) != -1)) {
            cout << "Camera: " << record.getCameraNum() << endl;
            cout << "Address: " << record.getAddress() << endl;
            cout << "Intersection: " << record.getIntersection() << endl;
            cout << "Neighborhood: " << record.getNeighborhoods() << endl;
            cout << endl;
            displayedCameras.push_back(record.getCameraNum());
            foundAny = true;
        }
    }
    if (!foundAny) {
        cout << "No cameras found." << endl;
    }
}



int main() {
    string filename;
    cout << "Enter file to use: " << endl;
    cin >> filename;

    // Reads the data from the file into records vector
    vector<CameraRecord> records = readFile(filename);
    if (records.empty()) {
        return 1;
    }
    
    int choice;
    // Menu to perform different operations for the user
    do {
        cout << "Select a menu option:" << endl;
        cout << "  1. Data overview" << endl;
        cout << "  2. Results by neighborhood" << endl;
        cout << "  3. Chart by month" << endl;
        cout << "  4. Search for cameras" << endl;
        cout << "  5. Exit" << endl;
        cout << "Your choice: " << endl;

        cin >> choice;

        // Handles menu options based on user's input
        switch (choice) {
            case 1:
                dataOverview(records);
                break;
            case 2:
                resultsByNeighborhood(records);
                break;
            case 3:
                chartByMonth(records);
                break;
            case 4: 
                searchCameras(records);
                break;
            case 5:
                break;
            default:
                break;
        }
        cout << endl;
    } while (choice != 5);  // Exits when user's choice is 5
}
