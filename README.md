# playground
Repository for the hello-world type of branches

Welcome to the playground. It's meant to play. So feel free to commit changes that call for other changes so both of us can play together.
Connect with me if you need permission. 

The full diagrams can be found on a [miro page](https://miro.com/app/board/uXjVKzGg22Y=/?userEmail=jnehme@liveintent.com&shareBoard=phillip@liveintent.com&track=true&utm_source=notification&utm_medium=email&utm_campaign=request-for-access&utm_content=open-sharing-settings&flow_feature=access_board&flow_type=request).

## Data Flow

The full data flow, starting with event triggering to the completion of the lambda function can be viewed in the picture below. There are 5 lambda functions that work together.
- **auth_handler** is triggered by a visit (or API call) to one of the auth page. auth_handle and handles interaction to save and validate credentials. it redirects to pages which retriger a aut_handler call for validation/success.
- **job_coordination** is scheduled daily. That lamba queries dynamo for each job, queries athena logs for entries that are in the config but are missing lctg values and saves these entries in an S3 bucket.
- **daily_maintenance** is triggered  by the job_coordination (if configured as such). It validates credentials daily and does housekeeping such as recreating fields on the ESP.
- **push_dispatcher** is triggered by files being written on the same S3 bucket which job coordination writes. push_dispatcher then converts each row of the files into events in a queue..
- **update_contact** which is triggered by the queue push_dispatcher created and updates the lcee field in the ESP.
  
![all_handlers in docs folder](https://github.com/LiveIntent/espresso/blob/main/docs/espresso_flow_all_handlers.png "Full flow")

## class structure
All handlers rely on the the main class BaseProcessor and its children. 
- BaseProcessor provides default implementation for daily maintenance called by the daily maintenance lambda, getting custome fields name, getting access tokens in the secret manager, processing auth_request used by the auth_handler lambad, as well as the abstract method definition for processing authentication, updating the lctg value and lctg updates used by the espresso_integration_processor lambda. It also provides basic work and helpers for authentication and API calls.
- There are three types of implementation of BaseProcessor depending on the authentication requirement.
  - APIKeyProcessor for ESPs that require a regular key for authorization.
  - OAouthProcessors for ESPs that require oAuth for authentication.
  - AuthProcessor for implementation that will not connect directly to ESP via Espresso. 
- Each ESP will then have its own subclass of one of the Auth subclass of BaseProcessor. For example.
  - Mailerlite as an example of APIKeyProcessor
  - drip  as an example of OAuthProcessor
  - Listrak is an example of an AuthProcessor which is used only for authentication and does not update the ESP. You can see it by the fact that it did not overwrite update_contact_lctg_value(). Applications outside Espresso then use this authentication.
![hierarchy png is in docs folder](https://github.com/LiveIntent/espresso/blob/main/docs/espresso_classes.png "Class hierarchy")


The daily Maintenance is scheduled daily
The auth are triggered via 
