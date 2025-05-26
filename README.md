# WorkFlow Delievery

This project demonstrates a food ordering workflow using Cadence. It processes orders from a CSV file and manages the order lifecycle through Cadence workflows.

## Runthrough of Cadence WorkFlow

<video controls width="100%" height="auto" playsinline autoplay muted loop>
  <source src="docs/assets/cadence-demo.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>

[View demo video on live site](https://selama206.github.io/cadence-teachback/#runthrough-of-cadence-workflow)

## Explain the process of your learning and implementation
I began by reviewing the official Cadence documentation to build a solid foundation in workflow concepts. To deepen my understanding, I explored [GitHub examples](https://github.com/cadence-workflow/cadence-java-samples) and tutorial videos from Cadence's YouTube channel. Before starting the main project, I built a simplified HelloWorld example to get hands-on experience with workflow architecture as code and to see how workflows show up in Cadence's UI. I also used AI tools throughout the process to speed up my learning and make debugging more efficient.


## Provide feedback about this process to explain how it can be improved
The Cadence YouTube videos linked on the [documentation page](https://cadenceworkflow.io/docs/get-started/video-tutorials) are visually clear and helpful to follow. There is a [video](https://www.youtube.com/watch?v=aLyRyNe5Ls0) not included in the video tutorials section that demonstrates how to register a domain on youtube which isn't currently linked on the Cadence page. Including that in the list would be a valuable addition for those setting up their environment and ramping up with Cadence/Workflow architecture for the first time.

## Explain how your workflow works in detail

### Workflow Components

### 1. Main Workflow (`HandleEatsOrderWorkflow`)

The main workflow orchestrates the entire food delivery process, acting as the conductor that:

- Makes business decisions (restaurant acceptance/rejection)
- Maintains order state across long periods
- Coordinates between different parts of the system
- Handles customer-facing order management
- Delegates delivery responsibilities
- Survives system failures without losing progress

The workflow logic flows like this:
```
Order Placed → Wait for Restaurant → [If Accepted] → Start Delivery → Track to Completion
                                  → [If Rejected] → End with Rejection
```

## WorkFlow Calls
Accepted Workflow:
```
"Your order is in front of your door!"
```
Rejected Workflow:
```
"Order UUID was rejected by the restaurant"
```


### 2. Child Workflow (`DeliverOrderWorkflow`)

The delivery workflow focuses solely on the delivery process and ensuring a response to the parent workflow. 


The delivery logic flows like this:
```
Start Delivery → Simulate Travel → Notify Delivery → Confirm Completion
```

### Activities Implementation

The activities are implemented in separate interfaces and classes for better modularity. Here are the key activities used in the project:

#### Order Processing Activities
1. **processOrder(String orderDetails)**
   - Used when an order is first received
   - Displays formatted order details including items
   - Called by the main workflow to acknowledge order receipt
   - Example: "Your order received! pizza, fries, soda"

2. **notifyOrderDelivered(String orderId)**
   - Called by the delivery workflow when delivery is complete
   - Sends delivery confirmation to the customer
   - Includes order ID for tracking
   - Example: "Order abc-123 delivered!"

3. **printDeliveryConfirmation(String orderId)**
   - Final confirmation of delivery
   - Provides delivery location information
   - Used as the last step in the delivery workflow
   - Example: "Your order is in front of your door!"

These activities are defined in the `EatsActivities` interface and implemented in `EatsActivityImpl`. They are registered with both the main worker and delivery worker to ensure consistent handling across the workflow.

### Workflow States and Transitions

#### Main Workflow States:
1. **Order Received**
   - Validates input parameters
   - Processes initial order details
   - Displays order confirmation

2. **Awaiting Restaurant Decision**
   - Waits for signal with 60-second timeout
   - Handles signal through `signalRestaurantDecision` method
   - Tracks decision state using CompletablePromise

3. **Order Processing**
   - If accepted: Simulates 3-second preparation time
   - If rejected: Returns rejection message
   - Handles errors with proper logging

4. **Delivery Initiation**
   - Creates child workflow with specific options
   - Sets appropriate timeouts and retry policies
   - Manages workflow ID generation

### Child Workflow States:
1. **Delivery Started**
   - Initializes delivery process
   - Logs start of delivery

2. **Delivery in Progress**
   - Simulates 4-second delivery time
   - Handles potential delivery issues

3. **Delivery Completed**
   - Sends delivery notifications
   - Confirms delivery completion
   - Returns success message

### Running WorkFlow

The project is containerized and processes orders from a CSV file. To run it:

1. Start the Cadence services :
In a separate terminal, clone Cadence repo to get a local version of Cadence service running on your machine. Then run the command:
```bash
 docker-compose up
```

2. In a seperate terminal start the worker to handle workflow tasks:
```bash
docker-compose build --no-cache worker && docker-compose up -d worker
```

3. In a seperate terminal process orders from the sample CSV:
```bash
docker-compose build --no-cache processor && docker-compose run processor sample_orders.csv
```

The script will read orders from the CSV, submit them to the workflow, and simulate restaurant decisions. Monitor the output to see the order status updates.
