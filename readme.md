# About
This is an example playbook and rulebook to perform a summarization of failed tasks in an Ansible Automation Platform (AAP) job by using an LLM, in this case OpenAI, to help summarize the failures.

Sometimes errors in Ansible can become hard to read such as python dumps or specific errors for an OS like Linux or Windows. This helps simplify the the error messages so that help desk and other users can easily understand why a job may have failed.

This setup consists of the following piece:
 - A notifier attached to a job template to notify EDA of a job failure
 - Rulebook to parse the job failure output
 - A playbook to parse the failures, send it to OpenAI, and then create a ServiceNow ticket with the failure info.

 Feel free to modify the prompt and tune it to your desired output.
<br>
<br>

# How it Works
When a job in AAP fails, and the notifier is attached, it will send a webhook to the EDA Event Stream. 

The rulebook has a simple conditional check to see if the status of the job was failed and to launch the specific Job Template (**modify this**) to parse data.

The job template uses the get_failed_jobs.yml playbook looks up the failed job data, parses it, sends it to OpenAI to summarize it and then finally creates a ServiceNow ticket with the summary.

<br>
<br>

# Steps to set up
This assumes you are using AAP 2.5 or at least the EDA 2.5 controller. 

To set up this integration, I chose to use a Basic Authentication Event Stream type in order to simplify the URL routing for the outbound alert.

- Create an [Event Stream](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.5/html/using_automation_decisions/simplified-event-routing#event-streams) in EDA controller to allow us a simplified URL to send the alerts to
- Create the [Rulebook Activation](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.5/html/using_automation_decisions/eda-rulebook-activations#eda-set-up-rulebook-activation) in EDA controller and use the Event Stream created above and the rulebook/rulebook.yml rulebook.
- Create a [Notifier](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.5/html/using_automation_execution/controller-notifications#controller-create-notification-template) of type Webhook and paste the Event Stream URL into the Target URL field.
![AAP Notifier](/images/notifier.png)

- Create a Job Template with the get_failed_jobs.yml as the playbook. You will need the following settings and credentials, depending on your setup
    - Check the box to "Prompt on launch" for extra variables, this allows us to pass in the EDA payload via the API.
    - Controller credential, to talk to the Controller API
    - OpenAI token 
    - ServiceNow credentials

- Attach the notifier to the job template and toggle "Failure".
![AAP Notifier](/images/jobtemplate.png)

<br>
<br>

# Playbook Requirements

You will need the following variables to run this playbook, modify as needed:

- **base_url**: Your AAP URL to concatenate with the job ID that will be parsed by the playbook
    - Example: "https://myaapurl.mycompany.com/api/controller/v2/jobs/"
- **snow_host**: Your ServiceNow instance URL
    - Example: "https://myinstance.service-now.com"
- **snow_user**: "admin"
- **snow_password**: "password" 
- **snow_caller**: The user that will be assigned as the creator of the ticket in ServiceNow
    - Example: "my_username"
- **openai_token**: Replace with the name of your LLM that you are using, but the Auth token for OpenAI

>If you are not already doing so, I strongly recommend creating [Custom Credential Types](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.5/html/using_automation_execution/assembly-controller-custom-credentials) to encrypt these variables.

<br>
<br>

# Rulebook Requirements

You will need to modify the job template name and organization name in the rulebook found in rulebooks/rulebooks.yml. You may also need to modify the webhook port.
<br>
<br>

# Example Output to ServiceNow
![SNOW Ticket](/images/snow_output.png)
