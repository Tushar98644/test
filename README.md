# test

package main

import "fmt"

func main() {
    fmt.Println("Hello, World!")
}



use std::net::TcpListener;
use std::io::{prelude::*, BufReader};
use std::fs;
use std::thread;
use std::time::Duration;
use threadpool::ThreadPool; // Requires 'threadpool' crate in Cargo.toml

fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();
    let pool = ThreadPool::new(4); // Create a thread pool with 4 threads

    for stream in listener.incoming().take(2) { // Handle up to 2 incoming connections for demonstration
        let stream = stream.unwrap();

        pool.execute(move || {
            handle_connection(stream);
        });
    }

    println!("Shutting down.");
}

fn handle_connection(mut stream: std::net::TcpStream) {
    let buf_reader = BufReader::new(&mut stream);
    let request_line = buf_reader.lines().next().unwrap().unwrap();

    let (status_line, filename) = match &request_line[..] {
        "GET / HTTP/1.1" => ("HTTP/1.1 200 OK", "hello.html"),
        "GET /sleep HTTP/1.1" => {
            thread::sleep(Duration::from_secs(5));
            ("HTTP/1.1 200 OK", "hello.html")
        }
        _ => ("HTTP/1.1 404 NOT FOUND", "404.html"),
    };

    let contents = fs::read_to_string(filename).unwrap();
    let length = contents.len();

    let response = format!(
        "{}\r\nContent-Length: {}\r\n\r\n{}",
        status_line, length, contents
    );

    stream.write_all(response.as_bytes()).unwrap();
}
