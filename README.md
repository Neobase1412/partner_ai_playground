# OpenAI Chat API Comparison: Python vs Rust

This project demonstrates how to build a simple backend server that interfaces with OpenAI's API using two different programming languages: Python and Rust. Each project implements the same functionality, allowing API callers to chat with OpenAI's GPT-4o-mini model.

## Python Implementation
### Python Project

1. **Create a virtual environment**:
   ```sh
   python -m venv venv
   ```

2. **Activate the virtual environment**:
   ```sh
   source venv/bin/activate
   ```

3. **Install required packages**:
   ```sh
   pip install fastapi uvicorn openai
   ```

4. **Create `main.py`**:
   ```python
   from fastapi import FastAPI, HTTPException
   from pydantic import BaseModel
   import openai

   app = FastAPI()

   class Message(BaseModel):
       content: str

   @app.post("/chat")
   async def chat(message: Message):
       try:
           response = openai.Completion.create(
               engine="text-davinci-003",
               prompt=message.content,
               max_tokens=150
           )
           return {"response": response.choices[0].text.strip()}
       except Exception as e:
           raise HTTPException(status_code=500, detail=str(e))

   if __name__ == "__main__":
       import uvicorn
       uvicorn.run(app, host="0.0.0.0", port=8000)
   ```

5. **Run the server**:
   ```sh
   uvicorn main:app --reload
   ```

### Rust Project

1. **Create a new Rust project**:
   ```sh
   cargo new openai-chat
   cd openai-chat
   ```

2. **Update `Cargo.toml`**:
   ```toml
   [dependencies]
   actix-web = "4"
   serde = { version = "1.0", features = ["derive"] }
   serde_json = "1.0"
   reqwest = { version = "0.11", features = ["json"] }
   ```

3. **Create `src/main.rs`**:
   ```rust
   use actix_web::{web, App, HttpServer, Responder, HttpResponse, post};
   use serde::Deserialize;
   use reqwest::Client;

   #[derive(Deserialize)]
   struct Message {
       content: String,
   }

   #[post("/chat")]
   async fn chat(message: web::Json<Message>) -> impl Responder {
       let client = Client::new();
       let res = client.post("https://api.openai.com/v1/engines/text-davinci-003/completions")
           .header("Authorization", "Bearer YOUR_OPENAI_API_KEY")
           .json(&serde_json::json!({
               "prompt": message.content,
               "max_tokens": 150
           }))
           .send()
           .await;

       match res {
           Ok(response) => {
               let text = response.text().await.unwrap_or_default();
               HttpResponse::Ok().body(text)
           },
           Err(_) => HttpResponse::InternalServerError().body("Error calling OpenAI API"),
       }
   }

   #[actix_web::main]
   async fn main() -> std::io::Result<()> {
       HttpServer::new(|| App::new().service(chat))
           .bind("0.0.0.0:8000")?
           .run()
           .await
   }
   ```

4. **Run the server**:
   ```sh
   cargo run
   ```