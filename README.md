# AI for the Church - Chat Completions API

The Chat Completions API is a powerful tool that generates responses based on a conversation context. This API uses Large Language Models to understand the context of a conversation and provide a relevant completion. This document outlines the specifications for making requests to the Chat Completions API endpoint.

## Endpoint

**POST** `http://api.aiforthechurch.org/inference/v1/chat/completions/`

## Authentication

Requests to this endpoint require a Bearer token in the Authorization header.

```
Authentication: Bearer <api token>
```

## Request Body

The request body must be a JSON object containing the following keys:

- `model` (string, required): The identifier for the AI model to use for generating completions.
- `messages` (array, required): An array of message objects that comprise the conversation history. Each message object must contain:
  - `role` (string, required): Indicates the role of the message sender. Valid values are `"system"`, `"user"`, or `"assistant"`.
  - `content` (string, required): The actual text content of the message.
- `temperature` (number, optional, nullable): Sampling temperature between 0 and 2. A higher value increases randomness while a lower value produces more deterministic results.
- `top_p` (number, optional, nullable): Nucleus sampling parameter, accepting values between 0 and 1. Defines the cumulative probability threshold at which token sampling is truncated.
- `top_k` (integer, optional, nullable): Limits the number of token candidates considered at each generation step, redistributing the probability mass among these candidates.
- `n` (integer, optional, nullable): Number of chat completion choices to generate for each input message.
- `frequency_penalty` (number, optional, nullable): Value between -2.0 and 2.0. Adjusts the likelihood of new tokens based on their current frequency in the text.
- `presence_penalty` (number, optional, nullable): Value between -2.0 and 2.0. Adjusts the likelihood of new tokens appearing based on their presence in the existing text.
- `stream` (boolean, optional, nullable): Indicates whether to stream back partial progress.
- `max_tokens` (integer, required): Maximum number of tokens to generate. Behavior upon limit exceedance is defined by `context_length_exceeded_behavior`.
- `stop` (array, optional): Up to 4 stop sequences where the API will cease further token generation.
- `prompt_truncate_len` (integer, optional, nullable): Length to which chat prompts should be truncated.
- `context_length_exceeded_behavior` (string, required): Behavior when the token count exceeds the model's context window. Options are `"truncate"` and `"error"`.

### Request Body Example

```json
{
  "model": "accounts/your_account/models/default",
  "messages": [
    {
      "role": "system",
      "content": "You are a helpful assistant."
    },
    {
      "role": "user",
      "content": "Thank you!"
    }
  ],
  "temperature": 0.7,
  "top_p": 1,
  "top_k": 50,
  "n": 1,
  "frequency_penalty": 0,
  "presence_penalty": 0,
  "stream": false,
  "max_tokens": 150,
  "stop": ["stop_sequence"],
  "prompt_truncate_len": null,
  "context_length_exceeded_behavior": "truncate"
}
```

## Chat Completions API Response

The response from the Chat Completions API contains detailed information about the generated chat completions, including metadata, the choices of completions, and usage statistics. This document describes the structure and content of a typical API response.

### Response Body

The response body is a JSON object containing the following fields:

- `id` (string, required): A unique identifier for the response.
- `object` (string, required): The type of the object, always `"chat.completion"` in this case.
- `created` (integer, required): The Unix timestamp indicating when the response was generated.
- `model` (string, required): The model used for the chat completion.
- `choices` (array of objects, required): An array of chat completion choices, each containing:
  - `index` (integer, required): The position of the chat completion choice in the list.
  - `message` (object, required): An object representing the generated message, with the following fields:
    - `role` (string, required): The role of the author of the message. Possible values are `"system"`, `"user"`, and `"assistant"`.
    - `content` (string, nullable): The text content of the message. This can be `null`.
  - `finish_reason` (string, required): The reason the generation was concluded. Possible values are `"stop"`, indicating a natural or defined stop point was reached, and `"length"`, indicating the max token count was hit.
- `usage` (object, required): An object containing statistics about the token usage in the request, including:
  - `prompt_tokens` (integer, required): The number of tokens in the prompt.
  - `completion_tokens` (integer, required): The number of tokens in the generated completion.
  - `total_tokens` (integer, required): The total number of tokens used in the request, including both the prompt and the completion.

For streaming responses, the `usage` field is included in the final response chunk returned. Please note that the presence of `usage` for streaming requests is specific to the OpenAI API, and direct access via the SDK might require handling the absence of this field in the type signature.

### Response Example

```json
{
  "id": "unique-id",
  "object": "chat.completion",
  "created": 1634492800,
  "model": "accounts/biblemate/models/default",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "You're welcome! How can I assist you further?"
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 15,
    "completion_tokens": 20,
    "total_tokens": 35
  }
}

### Handling the Response

Clients should check the HTTP status code of the response to determine if the request was successful. A 200 OK status indicates success, while a 4xx code denotes a client error (e.g., 400 Bad Request for invalid request syntax or 401 Unauthorized for authentication issues), and a 5xx status points to a server-side error.

The `choices` field contains the generated responses based on the input conversation context, and clients can select any choice for their next interaction. The `usage` field helps in tracking and analyzing the token usage for optimization and billing purposes.

## Rate Limits
Please note that there are rate limits to the number of requests you can make to the endpoint. Exceeding these limits may result in your API access being temporarily blocked.

For any further clarifications or support, please contact the API support team.
