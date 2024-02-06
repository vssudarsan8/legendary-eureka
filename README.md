# legendary-eureka
def assign_skus_to_machines_corrected(skus, machines):
    # Initialize the schedule and demand slack tracking
    schedule = {machine_name: [] for machine_name in machines}
    demand_slack = {sku: details['demand'] for sku, details in skus.items()}

    # Determine weeks when production should occur based on frequency
    production_weeks = {
        'monthly': [1],  # Choose the first week for monthly production
        'biweekly': [1, 3],  # First and third weeks for biweekly
        'weekly': [1, 2, 3, 4]  # Every week for weekly
    }

    # Step 1: Assign SKUs based on their frequency and batch size
    for sku, details in skus.items():
        for week in production_weeks[details['frequency']]:
            batch_size = details['demand']
            if details['frequency'] == 'biweekly':
                batch_size /= 2  # Divide total demand by 2 for biweekly
            elif details['frequency'] == 'monthly':
                batch_size /= 1  # Total demand for monthly

            assigned = False
            best_fit_machine = None
            best_fit_capacity_diff = float('inf')

            for machine_name, machine_details in machines.items():
                # Check capacity for the specific week
                capacity_diff = abs(machine_details['capacity'] - batch_size)
                if capacity_diff < best_fit_capacity_diff and machine_details['capacity'] >= batch_size:
                    best_fit_machine = machine_name
                    best_fit_capacity_diff = capacity_diff

            # Assign the SKU to the best fit machine for its production week
            if best_fit_machine is not None:
                schedule[best_fit_machine].append((sku, batch_size, week))
                machines[best_fit_machine]['capacity'] -= batch_size  # Update machine capacity
                demand_slack[sku] -= batch_size  # Update demand slack
                assigned = True

            if not assigned:
                demand_slack[sku] += batch_size  # Accumulate unmet demand

    # Step 2: Attempt to reduce demand slack by reallocating SKUs
    for sku, slack in demand_slack.items():
        if slack <= 0:
            continue  # Skip if no slack

        # Try to reallocate slack to underutilized machines
        for machine_name, machine_details in machines.items():
            if machine_details['capacity'] >= slack:
                schedule[machine_name].append((sku, slack, 'Reallocation'))
                machines[machine_name]['capacity'] -= slack
                demand_slack[sku] -= slack
                break

    # Output the adjusted schedule and demand slack
    print("Adjusted Schedule:")
    for machine, assignments in schedule.items():
        print(f"{machine}: {assignments}")
    print("\nAdjusted Demand Slack:")
    for sku, slack in demand_slack.items():
        print(f"{sku}: {slack}")

    return schedule, demand_slack

# Adjusted example SKUs and machines with corrected frequencies
skus = {
    'SKU1': {'demand': 100, 'frequency': 'monthly'},
    'SKU2': {'demand': 200, 'frequency': 'biweekly'},
    'SKU3': {'demand': 300, 'frequency': 'weekly'},
}
machines = {
    'Machine1': {'capacity': 500},
    'Machine2': {'capacity': 300},
}

# Run the adjusted heuristic
schedule, demand_slack = assign_skus_to_machines_corrected(skus, machines)
