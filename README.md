# Advanced-To-Do-List-Application 
#include <iostream>
#include <vector>
#include <string>
#include <algorithm>
#include <chrono>
#include <iomanip>
#include <fstream>
#include <sstream>

// Task Priority Levels
enum class Priority { LOW, MEDIUM, HIGH };

// Task Status
enum class Status { PENDING, IN_PROGRESS, COMPLETED };

// Task Class Definition
class Task {
private:
    static int nextId;  // Static counter for unique IDs
    int id;
    std::string description;
    Priority priority;
    Status status;
    std::chrono::system_clock::time_point dueDate;
    std::string category;

public:
    // Constructor
    Task(const std::string& desc, Priority prio, const std::string& cat, 
         const std::chrono::system_clock::time_point& due) 
        : description(desc), priority(prio), status(Status::PENDING),
          dueDate(due), category(cat) {
        id = ++nextId;
    }

    // Getters
    int getId() const { return id; }
    std::string getDescription() const { return description; }
    Priority getPriority() const { return priority; }
    Status getStatus() const { return status; }
    std::chrono::system_clock::time_point getDueDate() const { return dueDate; }
    std::string getCategory() const { return category; }

    // Setters
    void setStatus(Status newStatus) { status = newStatus; }
    void setDescription(const std::string& desc) { description = desc; }
    void setPriority(Priority prio) { priority = prio; }
    void setDueDate(const std::chrono::system_clock::time_point& due) { dueDate = due; }
    void setCategory(const std::string& cat) { category = cat; }

    // Utility functions
    std::string priorityToString() const {
        switch (priority) {
            case Priority::LOW: return "Low";
            case Priority::MEDIUM: return "Medium";
            case Priority::HIGH: return "High";
            default: return "Unknown";
        }
    }

    std::string statusToString() const {
        switch (status) {
            case Status::PENDING: return "Pending";
            case Status::IN_PROGRESS: return "In Progress";
            case Status::COMPLETED: return "Completed";
            default: return "Unknown";
        }
    }

    std::string dueDateToString() const {
        auto time = std::chrono::system_clock::to_time_t(dueDate);
        std::stringstream ss;
        ss << std::put_time(std::localtime(&time), "%Y-%m-%d");
        return ss.str();
    }
};

// Initialize static member
int Task::nextId = 0;

// TaskManager Class Definition
class TaskManager {
private:
    std::vector<Task> tasks;
    std::string filename;

    // Helper function to convert string to Priority
    Priority stringToPriority(const std::string& str) {
        if (str == "high" || str == "HIGH") return Priority::HIGH;
        if (str == "medium" || str == "MEDIUM") return Priority::MEDIUM;
        return Priority::LOW;
    }

public:
    TaskManager(const std::string& file = "tasks.txt") : filename(file) {
        loadTasks();
    }

    // Add a new task
    void addTask(const std::string& description, Priority priority, 
                const std::string& category, const std::chrono::system_clock::time_point& dueDate) {
        tasks.emplace_back(description, priority, category, dueDate);
        saveTasks();
    }

    // Remove a task by ID
    bool removeTask(int id) {
        auto it = std::find_if(tasks.begin(), tasks.end(),
            [id](const Task& task) { return task.getId() == id; });
        
        if (it != tasks.end()) {
            tasks.erase(it);
            saveTasks();
            return true;
        }
        return false;
    }

    // Update task status
    bool updateTaskStatus(int id, Status newStatus) {
        auto it = std::find_if(tasks.begin(), tasks.end(),
            [id](const Task& task) { return task.getId() == id; });
        
        if (it != tasks.end()) {
            it->setStatus(newStatus);
            saveTasks();
            return true;
        }
        return false;
    }

    // Display all tasks
    void displayTasks() const {
        if (tasks.empty()) {
            std::cout << "No tasks found.\n";
            return;
        }

        std::cout << std::setw(5) << "ID" << " | "
                  << std::setw(30) << "Description" << " | "
                  << std::setw(10) << "Priority" << " | "
                  << std::setw(12) << "Status" << " | "
                  << std::setw(12) << "Category" << " | "
                  << std::setw(12) << "Due Date" << "\n";
        std::cout << std::string(85, '-') << "\n";

        for (const auto& task : tasks) {
            std::cout << std::setw(5) << task.getId() << " | "
                      << std::setw(30) << task.getDescription() << " | "
                      << std::setw(10) << task.priorityToString() << " | "
                      << std::setw(12) << task.statusToString() << " | "
                      << std::setw(12) << task.getCategory() << " | "
                      << std::setw(12) << task.dueDateToString() << "\n";
        }
    }

    // Save tasks to file
    void saveTasks() const {
        std::ofstream file(filename);
        if (!file) {
            std::cerr << "Error: Could not open file for writing.\n";
            return;
        }

        for (const auto& task : tasks) {
            file << task.getId() << ","
                 << task.getDescription() << ","
                 << static_cast<int>(task.getPriority()) << ","
                 << static_cast<int>(task.getStatus()) << ","
                 << task.getCategory() << ","
                 << task.dueDateToString() << "\n";
        }
    }

    // Load tasks from file
    void loadTasks() {
        std::ifstream file(filename);
        if (!file) {
            return;  // File doesn't exist yet
        }

        std::string line;
        while (std::getline(file, line)) {
            std::stringstream ss(line);
            std::string item;
            std::vector<std::string> elements;

            while (std::getline(ss, item, ',')) {
                elements.push_back(item);
            }

            if (elements.size() == 6) {
                // Parse due date
                std::tm tm = {};
                std::stringstream ss(elements[5]);
                ss >> std::get_time(&tm, "%Y-%m-%d");
                auto dueDate = std::chrono::system_clock::from_time_t(std::mktime(&tm));

                // Create and add task
                Task task(elements[1], 
                         static_cast<Priority>(std::stoi(elements[2])),
                         elements[4],
                         dueDate);
                task.setStatus(static_cast<Status>(std::stoi(elements[3])));
                tasks.push_back(task);
            }
        }
    }

    // Filter tasks by category
    std::vector<Task> getTasksByCategory(const std::string& category) const {
        std::vector<Task> filteredTasks;
        std::copy_if(tasks.begin(), tasks.end(), std::back_inserter(filteredTasks),
            [&category](const Task& task) { return task.getCategory() == category; });
        return filteredTasks;
    }

    // Get overdue tasks
    std::vector<Task> getOverdueTasks() const {
        auto now = std::chrono::system_clock::now();
        std::vector<Task> overdueTasks;
        std::copy_if(tasks.begin(), tasks.end(), std::back_inserter(overdueTasks),
            [now](const Task& task) { 
                return task.getDueDate() < now && task.getStatus() != Status::COMPLETED; 
            });
        return overdueTasks;
    }
};

// Main function
int main() {
    TaskManager manager;
    std::string input;

    while (true) {
        std::cout << "\n=== To-Do List Manager ===\n"
                  << "1. Add Task\n"
                  << "2. Remove Task\n"
                  << "3. Update Task Status\n"
                  << "4. Display All Tasks\n"
                  << "5. Display Tasks by Category\n"
                  << "6. Display Overdue Tasks\n"
                  << "7. Exit\n"
                  << "Enter your choice: ";

        std::getline(std::cin, input);
        int choice = std::stoi(input);

        switch (choice) {
            case 1: {
                std::string description, category, priorityStr, dateStr;
                
                std::cout << "Enter task description: ";
                std::getline(std::cin, description);
                
                std::cout << "Enter priority (low/medium/high): ";
                std::getline(std::cin, priorityStr);
                
                std::cout << "Enter category: ";
                std::getline(std::cin, category);
                
                std::cout << "Enter due date (YYYY-MM-DD): ";
                std::getline(std::cin, dateStr);

                // Parse due date
                std::tm tm = {};
                std::stringstream ss(dateStr);
                ss >> std::get_time(&tm, "%Y-%m-%d");
                auto dueDate = std::chrono::system_clock::from_time_t(std::mktime(&tm));

                manager.addTask(description, manager.stringToPriority(priorityStr), 
                              category, dueDate);
                std::cout << "Task added successfully!\n";
                break;
            }
            case 2: {
                std::cout << "Enter task ID to remove: ";
                std::getline(std::cin, input);
                int id = std::stoi(input);
                
                if (manager.removeTask(id)) {
                    std::cout << "Task removed successfully!\n";
                } else {
                    std::cout << "Task not found.\n";
                }
                break;
            }
            case 3: {
                std::cout << "Enter task ID to update: ";
                std::getline(std::cin, input);
                int id = std::stoi(input);

                std::cout << "Enter new status (0:Pending, 1:In Progress, 2:Completed): ";
                std::getline(std::cin, input);
                Status newStatus = static_cast<Status>(std::stoi(input));

                if (manager.updateTaskStatus(id, newStatus)) {
                    std::cout << "Task status updated successfully!\n";
                } else {
                    std::cout << "Task not found.\n";
                }
                break;
            }
            case 4:
                manager.displayTasks();
                break;
            case 5: {
                std::string category;
                std::cout << "Enter category to filter by: ";
                std::getline(std::cin, category);
                
                auto filteredTasks = manager.getTasksByCategory(category);
                if (filteredTasks.empty()) {
                    std::cout << "No tasks found in this category.\n";
                } else {
                    for (const auto& task : filteredTasks) {
                        std::cout << "ID: " << task.getId() 
                                << ", Description: " << task.getDescription()
                                << ", Priority: " << task.priorityToString()
                                << ", Status: " << task.statusToString()
                                << ", Due: " << task.dueDateToString() << "\n";
                    }
                }
                break;
            }
            case 6: {
                auto overdueTasks = manager.getOverdueTasks();
                if (overdueTasks.empty()) {
                    std::cout << "No overdue tasks found.\n";
                } else {
                    std::cout << "Overdue Tasks:\n";
                    for (const auto& task : overdueTasks) {
                        std::cout << "ID: " << task.getId() 
                                << ", Description: " << task.getDescription()
                                << ", Due: " << task.dueDateToString() << "\n";
                    }
                }
                break;
            }
            case 7:
                std::cout << "Goodbye!\n";
                return 0;
            default:
                std::cout << "Invalid choice. Please try again.\n";
        }
    }
    return 0;
}
