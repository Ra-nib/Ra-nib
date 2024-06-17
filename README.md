#Project Title:CPU Schedular
#Project Description:A CPU scheduler is a crucial component of an operating
system responsible for managing the execution of processes. It decides which process
should utilize the CPU and for how long, ensuring efficient resource utilization. Through
algorithms and policies, it balances various factors like throughput, response time, and
fairness to optimize system performance. In essence, the CPU scheduler acts as a traffic
controller for the CPU, orchestrating the flow of processes for smooth operation.
#How to run the project:
// CPU scheduling Project 

#include <iostream>
#include <vector>
#include <queue>
#include <algorithm>
#include <iomanip>
#include "json.hpp" 
#include <fstream>

using namespace std;
using json = nlohmann::json;

// its like Classes only 
struct Process {
    int process_id;
    int arrival_time;
    int burst_time;
    int remaining_time;
    int priority;
};

// imput_function
void read_input(vector<Process>& processes) {
    int n;
    cin >> n;

    processes.resize(n);
    for (int i = 0; i < n; ++i) {
        cin >> processes[i].process_id >> processes[i].arrival_time >> processes[i].burst_time >> processes[i].priority;
        processes[i].remaining_time = processes[i].burst_time; // Initialize remaining time for Round Robin
    }
}

// Fn+ for_performance_metrics
void calculate_metrics(const vector<Process>& processes, const vector<int>& completion_times, float& avg_waiting_time, float& avg_turnaround_time) {
    int total_waiting_time = 0;
    int total_turnaround_time = 0;
    for (size_t i = 0; i < processes.size(); ++i) {
        int turnaround_time = completion_times[i] - processes[i].arrival_time;
        int waiting_time = turnaround_time - processes[i].burst_time;
        total_waiting_time += waiting_time;
        total_turnaround_time += turnaround_time;
    }
    avg_waiting_time = static_cast<float>(total_waiting_time) / processes.size();
    avg_turnaround_time = static_cast<float>(total_turnaround_time) / processes.size();
}

// Fn. for FCFS scheduling
json fcfs(const vector<Process>& processes) {
    int current_time = 0;
    int total_idle_time = 0;
    vector<int> completion_times(processes.size());
    vector<int> schedule;

    for (const auto& process : processes) {
        if (current_time < process.arrival_time) {
            total_idle_time += process.arrival_time - current_time;
            current_time = process.arrival_time;
        }
        for (int i = 0; i < process.burst_time; ++i) {
            schedule.push_back(process.process_id);
        }
        current_time += process.burst_time;
        completion_times[process.process_id - 1] = current_time;
    }

    float avg_waiting_time, avg_turnaround_time;
    calculate_metrics(processes, completion_times, avg_waiting_time, avg_turnaround_time);
    float cpu_utilization = (static_cast<float>(current_time - total_idle_time) / current_time) * 100;
    float throughput = static_cast<float>(processes.size()) / current_time;

    json result;
    result["schedule"] = schedule;
    result["avg_waiting_time"] = avg_waiting_time;
    result["avg_turnaround_time"] = avg_turnaround_time;
    result["cpu_utilization"] = cpu_utilization;
    result["throughput"] = throughput;

    return result;
}

// Fn. for SJN scheduling
json sjn(vector<Process> processes) {
    int current_time = 0;
    int total_idle_time = 0;
    vector<int> completion_times(processes.size());
    vector<int> schedule;

    auto cmp = [](const Process& a, const Process& b) { return a.burst_time > b.burst_time; };
    priority_queue<Process, vector<Process>, decltype(cmp)> pq(cmp);

    int i = 0;
    while (i < processes.size() || !pq.empty()) {
        while (i < processes.size() && processes[i].arrival_time <= current_time) {
            pq.push(processes[i]);
            ++i;
        }
        if (!pq.empty()) {
            Process current_process = pq.top();
            pq.pop();
            for (int j = 0; j < current_process.burst_time; ++j) {
                schedule.push_back(current_process.process_id);
            }
            current_time += current_process.burst_time;
            completion_times[current_process.process_id - 1] = current_time;
        } else {
            total_idle_time++;
            ++current_time;
        }
    }

    float avg_waiting_time, avg_turnaround_time;
    calculate_metrics(processes, completion_times, avg_waiting_time, avg_turnaround_time);
    float cpu_utilization = (static_cast<float>(current_time - total_idle_time) / current_time) * 100;
    float throughput = static_cast<float>(processes.size()) / current_time;

    json result;
    result["schedule"] = schedule;
    result["avg_waiting_time"] = avg_waiting_time;
    result["avg_turnaround_time"] = avg_turnaround_time;
    result["cpu_utilization"] = cpu_utilization;
    result["throughput"] = throughput;

    return result;
}

// Fn, for Round Robin scheduling
json round_robin(vector<Process> processes, int quantum) {
    int current_time = 0;
    int total_idle_time = 0;
    vector<int> completion_times(processes.size());
    vector<int> schedule;
    queue<Process> rq;

    int i = 0;
    while (i < processes.size() || !rq.empty()) {
        while (i < processes.size() && processes[i].arrival_time <= current_time) {
            rq.push(processes[i]);
            ++i;
        }
        if (!rq.empty()) {
            Process current_process = rq.front();
            rq.pop();
            int time_slice = min(quantum, current_process.remaining_time);
            for (int j = 0; j < time_slice; ++j) {
                schedule.push_back(current_process.process_id);
            }
            current_time += time_slice;
            current_process.remaining_time -= time_slice;
            if (current_process.remaining_time > 0) {
                while (i < processes.size() && processes[i].arrival_time <= current_time) {
                    rq.push(processes[i]);
                    ++i;
                }
                rq.push(current_process);
            } else {
                completion_times[current_process.process_id - 1] = current_time;
            }
        } else {
            total_idle_time++;
            ++current_time;
        }
    }

    float avg_waiting_time, avg_turnaround_time;
    calculate_metrics(processes, completion_times, avg_waiting_time, avg_turnaround_time);
    float cpu_utilization = (static_cast<float>(current_time - total_idle_time) / current_time) * 100;
    float throughput = static_cast<float>(processes.size()) / current_time;

    json result;
    result["schedule"] = schedule;
    result["avg_waiting_time"] = avg_waiting_time;
    result["avg_turnaround_time"] = avg_turnaround_time;
    result["cpu_utilization"] = cpu_utilization;
    result["throughput"] = throughput;

    return result;
}

// Fn. for  Priority scheduling
json priority_scheduling(vector<Process> processes) {
    int current_time = 0;
    int total_idle_time = 0;
    vector<int> completion_times(processes.size());
    vector<int> schedule;

    auto cmp = [](const Process& a, const Process& b) { return a.priority > b.priority; };
    priority_queue<Process, vector<Process>, decltype(cmp)> pq(cmp);

    int i = 0;
    while (i < processes.size() || !pq.empty()) {
        while (i < processes.size() && processes[i].arrival_time <= current_time) {
            pq.push(processes[i]);
            ++i;
        }
        if (!pq.empty()) {
            Process current_process = pq.top();
            pq.pop();
            for (int j = 0; j < current_process.burst_time; ++j) {
                schedule.push_back(current_process.process_id);
            }
            current_time += current_process.burst_time;
            completion_times[current_process.process_id - 1] = current_time;
        } else {
            total_idle_time++;
            ++current_time;
        }
    }

    float avg_waiting_time, avg_turnaround_time;
    calculate_metrics(processes, completion_times, avg_waiting_time, avg_turnaround_time);
    float cpu_utilization = (static_cast<float>(current_time - total_idle_time) / current_time) * 100;
    float throughput = static_cast<float>(processes.size()) / current_time;

    json result;
    result["schedule"] = schedule;
    result["avg_waiting_time"] = avg_waiting_time;
    result["avg_turnaround_time"] = avg_turnaround_time;
    result["cpu_utilization"] = cpu_utilization;
    result["throughput"] = throughput;

    return result;
}

// fn. writing scheduling data to JSON file
void write_to_json(const std::string& filename, const json& data) {
    std::ofstream file(filename);
    if (file.is_open()) {
        file << data.dump(4); // Pretty-print with 4-space indentation
        file.close();
    } else {
        std::cerr << "Unable to open file: " << filename << std::endl;
    }
}

int main() {
    vector<Process> processes;
    read_input(processes);

    // Sorting processes by arrival time before we start
    sort(processes.begin(), processes.end(), [](const Process& a, const Process& b) {
        return a.arrival_time < b.arrival_time;
    });

    json output;
    output["fcfs"] = fcfs(processes);
    output["sjn"] = sjn(processes);
    int quantum;
    cin >> quantum;
    output["round_robin"] = round_robin(processes, quantum);
    output["priority"] = priority_scheduling(processes);

    // Writing JSON data to thw followng file ,,,
    // plz note that this file will be found in the same folder 
    write_to_json("scheduling_results.json", output);

    return 0;
}



#Internal working of Project:
#What i have learnt:
#Resources:
