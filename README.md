# Shiva-
Project 

package com.example.webhook;

import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.http.*;
import org.springframework.web.client.RestTemplate;
import java.util.HashMap;
import java.util.Map;

@SpringBootApplication
public class SpringWebhookApp implements CommandLineRunner {

    private final RestTemplate restTemplate = new RestTemplate();

    public static void main(String[] args) {
        SpringApplication.run(SpringWebhookApp.class, args);
    }

    @Override
    public void run(String... args) throws Exception {
        generateWebhookAndSendQuery();
    }

    private void generateWebhookAndSendQuery() {
        String url = "https://bfhldevapigw.healthrx.co.in/hiring/generateWebhook/JAVA";

        // Step 1: Request Body
        Map<String, Object> body = new HashMap<>();
        body.put("name", "John Doe");
        body.put("regNo", "REG12347");  // Replace with your actual regNo
        body.put("email", "john@example.com");

        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_JSON);
        HttpEntity<Map<String, Object>> request = new HttpEntity<>(body, headers);

        // Step 2: Send POST Request
        ResponseEntity<Map> response = restTemplate.postForEntity(url, request, Map.class);

        System.out.println("Response: " + response.getBody());

        String webhookUrl = (String) response.getBody().get("webhook");
        String accessToken = (String) response.getBody().get("accessToken");

        // Step 3: Prepare SQL Query (Example)
        String finalQuery = "SELECT * FROM employees WHERE salary > 50000;";

        // Step 4: Send SQL Query with JWT Authorization
        sendFinalQuery(webhookUrl, accessToken, finalQuery);
    }

    private void sendFinalQuery(String webhookUrl, String accessToken, String sqlQuery) {
        Map<String, Object> payload = new HashMap<>();
        payload.put("finalQuery", sqlQuery);

        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_JSON);
        headers.set("Authorization", accessToken);

        HttpEntity<Map<String, Object>> entity = new HttpEntity<>(payload, headers);
        ResponseEntity<String> result = restTemplate.postForEntity(webhookUrl, entity, String.class);

        System.out.println("Webhook Response: " + result.getBody());
    }
}
