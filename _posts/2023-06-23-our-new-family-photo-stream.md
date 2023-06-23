---
layout: post
title: Our New Family Photo Stream
date: 2023-06-23 10:31 -0400
description: Building a cost-effective and efficient solution for a family photo stream, leveraging AWS services like Lambda, S3, and API Gateway. We detailed the process of server-side and client-side pagination, and how these components work together to provide seamless photo streaming, with the complete code available on GitHub for further exploration.
---
Hello folks! I've recently transitioned my family photo stream from a setup involving PhotoPrism hosted on Digital Ocean (costing around $60/month), to a solution employing AWS services. This journey started as a professional venture where I experimented with AWS Lambdas and API Gateway. As it turns out, these technologies are quite handy in not just streamlining backend operations, but also driving down costs.

## AWS Lambdas and API Gateway: The Dynamic Duo

AWS Lambda is a compute service that lets you run code without worrying about servers. In tandem with API Gateway, you can build a serverless backend, which can operate based on event triggers. For our case, the event was the request for photos from my frontend, which would then invoke the Lambda function through the API Gateway.

API Gateway acts as a traffic conductor, routing requests, transforming protocols, keeping tabs on calls, and ensuring secure connectivity between your backend services and APIs exposed to your app.

## Joining Forces: Lambda, S3 and API Gateway

The central architecture of my app revolves around AWS Lambda, S3, and API Gateway.

Photos are stored in an S3 bucket, thereby converting it into a photo database. I zeroed in on S3 for its cost-efficiency, high durability, and scalable nature, making it ideal for storing a vast volume of data such as photos. However, utilizing S3 as a database presented a unique set of challenges, particularly in terms of sorting and indexing files.

When the frontend requests photos, an AWS Lambda function is triggered. This interaction is facilitated by the API Gateway. The Lambda function fetches the photos (in the form of presigned URLs for security) from the S3 bucket and forwards them to the frontend. Crucially, the Lambda function also manages pagination, employing the 'list_objects_v2' function from the AWS SDK and a 'pageToken' from the request to fetch the next batch of photos.

## Exploring the Code

### Server-Side Pagination with Lambda

The key to the photo stream's smooth operation lies in our AWS Lambda function. Within this function, we handle pagination, allowing us to retrieve photos from our S3 bucket in a manageable, piece-by-piece manner.

In the `lambda_handler` function, we extract the `page_token` from the event object sent by the client. We then pass this `page_token` along with the desired `page_size` (the number of items to retrieve per page) to the `list_objects_v2` method:

```ruby
def lambda_handler(event:, context:)
  ...
  page_size = 10  # Number of items to retrieve per page
  page_token = extract_page_token(event)
  page_token = page_token.empty? ? nil : page_token

  # Retrieve photos with pagination
  response = s3_client.list_objects_v2(bucket: bucket_name, max_keys: page_size, continuation_token: page_token)
  ...
end
```

The `list_objects_v2` method, part of the AWS SDK for Ruby, communicates with our S3 bucket to retrieve the specified page of photos. By using the `continuation_token` parameter, which corresponds to our `page_token`, we keep track of where we left off in the list of photos on S3. This enables our pagination functionality.

### Client-Side Integration

Our front-end, which you can see in action at [https://photos.lennonbaird.com/](https://photos.lennonbaird.com/), interacts with this Lambda function to fetch and display the photos.

Upon loading the page or clicking the "Load More" button, the `loadPhotos` JavaScript function is triggered:

```javascript
function loadPhotos() {
    const pageToken = $('#pageToken').val();

    $.post(process.env.API_GATEWAY_URL, { pageToken: pageToken }, function(data) {
        const response = JSON.parse(data);
        const items = response.items;
        const nextPageToken = response.nextPageToken;

        items.forEach(function(photo) {
            ...
        });

        if (nextPageToken) {
            $('#pageToken').val(nextPageToken || '');
            $('#loadMoreBtn').show();
        } else {
            $('#pageToken').val('');
            $('#loadMoreBtn').hide();
        }
    });
}
```

This function posts a request to our API Gateway (represented here as `process.env.API_GATEWAY_URL`) with the current `pageToken`. This token is then processed on the server-side as we discussed earlier.

Once we receive the response from the server, we add the new photos to our page and update the `pageToken` for the next request. This operation enables seamless client-side pagination and a smooth user experience.

### Dive Deeper

If you're interested in the specifics, or you want to try deploying your own version, I invite you to explore the code on GitHub [https://github.com/jeffreybaird/aws-photo-stream](https://github.com/jeffreybaird/aws-photo-stream). You'll find all the details on how the different parts of the system interact to provide a cost-effective and efficient photo stream.

## Concluding Thoughts

Creating this photo stream was an excellent opportunity to delve deeper into several AWS services such as AWS Lambda, S3, and API Gateway, and how they can work together. While the road had its challenges, it also provided a rich learning experience and resulted in a successful and cost-effective project.

This architecture worked for my use-case, and I thoroughly enjoyed putting all the pieces together. I hope this walkthrough inspires anyone looking to leverage these AWS services for a similar project.

Enjoy exploring the photo stream and the code. I'd love to hear your thoughts or answer any questions. Feel free to drop your comments on GitHub. Happy coding!