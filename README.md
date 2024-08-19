# playground
Repository for the hello-world type of branches

Welcome to the playground. It's meant to play. So feel free to commit changes that call for other changes so both of us can play together.
Connect with me if you need permission. 

The full diagrams can be found on a [miro page](https://miro.com/app/board/uXjVKzGg22Y=/?userEmail=jnehme@liveintent.com&shareBoard=phillip@liveintent.com&track=true&utm_source=notification&utm_medium=email&utm_campaign=request-for-access&utm_content=open-sharing-settings&flow_feature=access_board&flow_type=request).

## Data Flow

The full data flow, from event triggering can be seen is the picture below. There are 5 lambda functions that work together.
- **auth_handler** which is triggered by a visit (or API call) to one of the auth page and handles web interaction as well as validating and saving the credentials. Auth_handler redirects to pages which retriger a aut_handler call for validation/success.
- **job_coordination** which is triggered daily, queries for each configuration and saves the results in S3 buket.
- **daily_maintenance** which is triggered (if configured as such) by the job_coordination, validates credentials daily and does housekeeping such as recreating fields on the ESP
- **push_dispatcher** which is triggered by files being written on the same S3 bucket which job coordination writes. Push dispatcher then converts the files into queued events.
- **update_contact** which is triggered by the queue push_dispatcher created and updates the lcee field in the ESP.
  
![all_handlers in docs folder](https://github.com/LiveIntent/espresso/blob/main/docs/espresso_flow_all_handlers.png "Full flow")

## class structure
All handlers rely on the the main class hierarchy for the BaseProcessor.  BaseProcessor provides default implementation for daily maintenance called by the daily maintenance lambda, getting custome fields name, getting access tokens in the secret manager, processing auth_request used by the auth_handler lambad, as well as the abstract method definition for processing authentication, updating the lctg value and lctg updates used by the espresso_integration_processor lambda. It also provides basic work and helpers for authentication and API calls. 
![hierarchy png is in docs folder](https://github.com/LiveIntent/espresso/blob/main/docs/espresso_classes.png "Class hierarchy")

The daily Maintenance is scheduled daily
The auth are triggered via 
