from jira import JIRA

# API-key
api_token = "API_KEY_JIRA"
PROJECT_KEY = 'YOUR_PROJECT_KEY'
issue_types = ['Story', 'Task', 'Bug']

options = {'server': 'SERVER'}
jira = JIRA(options, basic_auth=('EMAIL', api_token))

def count_bugs_by_developer():
    # Construct JQL query to search for tasks or stories in the specified project
    jql_query = f"project = '{PROJECT_KEY}' AND issuetype IN ({', '.join(issue_types)}) AND assignee IS NOT EMPTY"

    # Get a list of tasks or stories
    max_results = 100
    start_at = 0
    total_issues = []
    while True:
        issues = jira.search_issues(jql_query, startAt=start_at, maxResults=max_results)
        total_issues.extend(issues)
        if len(issues) < max_results:
            break
        start_at += max_results

    # Dictionary to count bugs by developers
    developer_bug_count = {}

    # Iterate over the issues
    for issue in total_issues:
        # Get the list of related bugs (Bug sub-task)
        sub_tasks = jira.search_issues(f"parent = '{issue.key}' AND issuetype = 'Bug sub-task'")

        # Iterate over the bugs and increment the count for each developer
        for sub_task in sub_tasks:
            developer = sub_task.fields.assignee

            # Increment the bug count for the developer
            developer_bug_count[developer] = developer_bug_count.get(developer, 0) + 1

        # Output intermediate results for each issue
        print(f"Issue: {issue.key} | Number of related bugs: {len(sub_tasks)}")

    # Output the bug count by developer
    print("\nBug count by developer:")
    print("Developer | Bug Count")
    for developer, bug_count in developer_bug_count.items():
        print(f"{developer} | {bug_count}")

# Run the script
count_bugs_by_developer()
