#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>

#define PORT 9999
#define BUFFER_SIZE 1024

int main() {
    int server_fd, new_sock;
    struct sockaddr_in server_addr, client_addr;
    socklen_t addr_len = sizeof(client_addr);
    char buffer[BUFFER_SIZE];
    server_fd = socket(AF_INET, SOCK_STREAM, 0);
    if (server_fd == -1) {
        perror("Socket creation failed");
        exit(EXIT_FAILURE);
    }

    server_addr.sin_family = AF_INET;
    server_addr.sin_addr.s_addr = INADDR_ANY;
    server_addr.sin_port = htons(PORT);
    if (bind(server_fd, (struct sockaddr*)&server_addr, sizeof(server_addr)) < 0) {
        perror("Bind failed");
        close(server_fd);
        exit(EXIT_FAILURE);
    }

    if (listen(server_fd, 3) < 0) {
        perror("Listen failed");
        close(server_fd);
        exit(EXIT_FAILURE);
    }

    printf("Server listening on port %d...\n", PORT);

    new_sock = accept(server_fd, (struct sockaddr*)&client_addr, &addr_len);
    if (new_sock < 0) {
        perror("Accept failed");
        close(server_fd);
        exit(EXIT_FAILURE);
    }

    printf("Client connected! Start chatting (type 'exit' to quit).\n\n");

    while (1) {
        memset(buffer, 0, BUFFER_SIZE);
        ssize_t bytes_received = recv(new_sock, buffer, BUFFER_SIZE - 1, 0);
        if (bytes_received <= 0 || strncmp(buffer, "exit", 4) == 0) {
            printf("Client disconnected.\n");
            break;
        }
        printf("Client: %s", buffer);
        printf("Server: ");
        fgets(buffer, BUFFER_SIZE, stdin);
        send(new_sock, buffer, strlen(buffer), 0);
        if (strncmp(buffer, "exit", 4) == 0) {
            printf("Ending chat...\n");
            break;
        }
    }

    close(new_sock);
    close(server_fd);
    return 0;
}

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>

#define PORT 5555
#define BUFFER_SIZE 1024

void handle_retrieve(int client_socket, const char *filename)
{
    char buffer[BUFFER_SIZE];
    FILE *fp = fopen(filename, "rb");

    if (fp == NULL)
    {
        printf("File '%s' not found on server.\n", filename);
        send(client_socket, "ERROR: File not found", 21, 0);
        return;
    }

    send(client_socket, "OK", 2, 0);
    usleep(10000);

    printf("\nSending file '%s' to client...\n", filename);

    int bytes_read;

    while ((bytes_read = fread(buffer, 1, BUFFER_SIZE, fp)) > 0)
    {
        send(client_socket, buffer, bytes_read, 0);
        memset(buffer, 0, BUFFER_SIZE);
    }

    fclose(fp);

    printf("File '%s' sent successfully!\n", filename);
}

void handle_store(int client_socket, const char *filename)
{
    char buffer[BUFFER_SIZE];
    char save_name[BUFFER_SIZE + 30];

    snprintf(save_name, sizeof(save_name), "uploaded_%s", filename);

    FILE *fp = fopen(save_name, "wb");

    if (fp == NULL)
    {
        perror("Failed to create file on server");
        send(client_socket, "ERROR: Server storage failure", 29, 0);
        return;
    }

    send(client_socket, "OK", 2, 0);

    printf("\nReceiving file from client...\n");
    printf("Saving as '%s'...\n", save_name);

    int bytes_received;

    while ((bytes_received = recv(client_socket,
                                  buffer,
                                  BUFFER_SIZE,
                                  0)) > 0)
    {
        if (strncmp(buffer, "EOF_SIGNAL", 10) == 0)
            break;

        fwrite(buffer, 1, bytes_received, fp);
        memset(buffer, 0, BUFFER_SIZE);
    }

    fclose(fp);

    printf("File saved successfully as '%s'.\n", save_name);

    printf("\n========== UPLOADED FILE CONTENT ==========\n");

    fp = fopen(save_name, "rb");

    if (fp == NULL)
    {
        printf("Unable to open uploaded file.\n");
        return;
    }

    int total_bytes = 0;

    while ((bytes_received = fread(buffer, 1, BUFFER_SIZE, fp)) > 0)
    {
        fwrite(buffer, 1, bytes_received, stdout);
        total_bytes += bytes_received;
    }

    fclose(fp);

    if (total_bytes == 0)
    {
        printf("There is no content in the file.\n");
    }

    printf("\n========== END OF FILE ==========\n");
}

int main()
{
    int server_fd, client_socket;
    struct sockaddr_in address;
    int addrlen = sizeof(address);

    char command[BUFFER_SIZE];
    char filename[BUFFER_SIZE];

    if ((server_fd = socket(AF_INET, SOCK_STREAM, 0)) == 0)
    {
        perror("Socket creation failed");
        exit(EXIT_FAILURE);
    }

    int opt = 1;

    setsockopt(server_fd,
               SOL_SOCKET,
               SO_REUSEADDR,
               &opt,
               sizeof(opt));

    address.sin_family = AF_INET;
    address.sin_addr.s_addr = INADDR_ANY;
    address.sin_port = htons(PORT);

    if (bind(server_fd,
             (struct sockaddr *)&address,
             sizeof(address)) < 0)
    {
        perror("Bind failed");
        close(server_fd);
        exit(EXIT_FAILURE);
    }

    if (listen(server_fd, 5) < 0)
    {
        perror("Listen failed");
        close(server_fd);
        exit(EXIT_FAILURE);
    }

    printf("Iterative TCP FTP Server running on port %d...\n", PORT);

    while (1)
    {
        printf("\n---------------------------------------------\n");
        printf("Server ready. Waiting for connection...\n");

        if ((client_socket = accept(server_fd,
                                    (struct sockaddr *)&address,
                                    (socklen_t *)&addrlen)) < 0)
        {
            perror("Accept failed");
            continue;
        }

        printf("Client connected!\n");

        while (1)
        {
            memset(command, 0, BUFFER_SIZE);
            memset(filename, 0, BUFFER_SIZE);

            int bytes_read = recv(client_socket,
                                  command,
                                  BUFFER_SIZE - 1,
                                  0);

            if (bytes_read <= 0 ||
                strncmp(command, "EXIT", 4) == 0)
            {
                printf("Client requested disconnect or connection closed.\n");
                break;
            }

            command[strcspn(command, "\r\n")] = 0;

            if (strncmp(command, "RETRIEVE", 8) == 0)
            {
                recv(client_socket,
                     filename,
                     BUFFER_SIZE - 1,
                     0);

                filename[strcspn(filename, "\r\n")] = 0;

                printf("Client requested RETRIEVE for file: '%s'\n",
                       filename);

                handle_retrieve(client_socket, filename);
            }
            else if (strncmp(command, "STORE", 5) == 0)
            {
                recv(client_socket,
                     filename,
                     BUFFER_SIZE - 1,
                     0);

                filename[strcspn(filename, "\r\n")] = 0;

                printf("Client requested STORE for file: '%s'\n",
                       filename);

                handle_store(client_socket, filename);
            }
        }

        close(client_socket);

        printf("Connection closed. Waiting for next client...\n");
    }

    close(server_fd);

    return 0;
}
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>

#define PORT 8888
#define BUFFER_SIZE 1024
#define HASH_SIZE 10

struct Node {
    char ip[32];
    char mac[32];
    struct Node* next;
};

struct Node* hashTable[HASH_SIZE];

unsigned int hash_function(const char* ip) {
    unsigned int hash = 0;
    while (*ip) {
        hash = (hash * 31) + *ip++;
    }
    return hash % HASH_SIZE;
}

void insert_arp(const char* ip, const char* mac) {
    unsigned int index = hash_function(ip);
    struct Node* newNode = (struct Node*)malloc(sizeof(struct Node));
    strcpy(newNode->ip, ip);
    strcpy(newNode->mac, mac);
    newNode->next = hashTable[index];
    hashTable[index] = newNode;
}

char* lookup_arp(const char* ip) {
    unsigned int index = hash_function(ip);
    struct Node* current = hashTable[index];
    while (current != NULL) {
        if (strcmp(current->ip, ip) == 0) {
            return current->mac;
        }
        current = current->next;
    }
    return NULL;
}

void load_system_arp_cache() {
    for (int i = 0; i < HASH_SIZE; i++) hashTable[i] = NULL;

    FILE *fp = popen("ip neigh | awk '/lladdr/ {print $1, $5}'", "r");
    if (fp == NULL) {
        printf(" Shell command execution failed.\n");
        return;
    }

    char ip[32], mac[32];
    int count = 0;
    while (fscanf(fp, "%31s %31s", ip, mac) == 2) {
        insert_arp(ip, mac);
        printf("  [Shell -> Hash Entry] IP: %-15s -> MAC: %s\n", ip, mac);
        count++;
    }
    pclose(fp);

    if (count == 0) {
        printf("  [ System ARP Cache empty. Fallback localhost entry loaded...\n");
        insert_arp("127.0.0.1", "00:00:00:00:00:00");
    }
}

int main() {
    int server_fd, client_socket;
    struct sockaddr_in address;
    int addrlen = sizeof(address);
    char buffer[BUFFER_SIZE];
    int opt = 1;

    printf(" Executing Shell Command 'ip neigh' to build Hash Table...\n");
    load_system_arp_cache();

    server_fd = socket(AF_INET, SOCK_STREAM, 0);
    setsockopt(server_fd, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt));

    address.sin_family = AF_INET;
    address.sin_addr.s_addr = INADDR_ANY;
    address.sin_port = htons(PORT);

    bind(server_fd, (struct sockaddr *)&address, sizeof(address));
    listen(server_fd, 5);

    printf("\n ARP Server running on port %d...\n", PORT);

    while (1) {
        client_socket = accept(server_fd, (struct sockaddr *)&address, (socklen_t*)&addrlen);
        if (client_socket < 0) continue;

        printf("\nClient Connected!\n");

        while (1) {
            memset(buffer, 0, BUFFER_SIZE);
            int bytes_read = recv(client_socket, buffer, BUFFER_SIZE - 1, 0);

            if (bytes_read <= 0 || strncmp(buffer, "exit", 4) == 0 || strncmp(buffer, "bye", 3) == 0) {
                printf("Client Disconnected.\n");
                break;
            }

            buffer[strcspn(buffer, "\r\n")] = 0;
            printf(" ARP Query IP: %s\n", buffer);

            char* mac = lookup_arp(buffer);
            if (mac != NULL) {
                printf("Hash Match Found: %s\n", mac);
                send(client_socket, mac, strlen(mac), 0);
            } else {
                printf("IP Not Found in Hash Table.\n");
                send(client_socket, "ERROR: MAC Not Found in Hash Table", 34, 0);
            }
        }

        close(client_socket);
    }

    close(server_fd);
    return 0;
}

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>

#define SERVER_IP "127.0.0.1"
#define PORT 9999
#define BUFFER_SIZE 1024

int main() {
    int sockfd;
    struct sockaddr_in server_addr;
    char buffer[BUFFER_SIZE];

    sockfd = socket(AF_INET, SOCK_STREAM, 0);
    if (sockfd < 0) {
        perror("Socket creation failed");
        exit(EXIT_FAILURE);
    }

    server_addr.sin_family = AF_INET;
    server_addr.sin_port = htons(PORT);
    inet_pton(AF_INET, SERVER_IP, &server_addr.sin_addr);

    if (connect(sockfd, (struct sockaddr*)&server_addr, sizeof(server_addr)) < 0) {
        perror("Connection failed");
        close(sockfd);
        exit(EXIT_FAILURE);
    }

    printf("Connected to Server! Start chatting (type 'exit' to quit).\n\n");

    while (1) {
        memset(buffer, 0, BUFFER_SIZE);
        printf("Client: ");
        fgets(buffer, BUFFER_SIZE, stdin);
        send(sockfd, buffer, strlen(buffer), 0);
        if (strncmp(buffer, "exit", 4) == 0) {
            printf("Ending chat...\n");
            break;
        }
        memset(buffer, 0, BUFFER_SIZE);
        ssize_t bytes_received = recv(sockfd, buffer, BUFFER_SIZE - 1, 0);
        if (bytes_received <= 0 || strncmp(buffer, "exit", 4) == 0) {
            printf("Server disconnected.\n");
            break;
        }
        printf("Server: %s", buffer);
    }

    close(sockfd);
    return 0;
}
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>

#define PORT 5555
#define BUFFER_SIZE 1024

void store_file(int sock)
{
    char filename[BUFFER_SIZE];
    char buffer[BUFFER_SIZE];

    printf("Enter local filename to store on server (upload): ");

    fgets(filename, BUFFER_SIZE, stdin);
    filename[strcspn(filename, "\r\n")] = 0;

    FILE *fp = fopen(filename, "rb");

    if (fp == NULL)
    {
        printf("Local file '%s' does not exist!\n", filename);
        return;
    }

    send(sock, "STORE", 5, 0);

    usleep(10000);

    send(sock, filename, strlen(filename), 0);

    memset(buffer, 0, BUFFER_SIZE);

    recv(sock, buffer, BUFFER_SIZE - 1, 0);

    if (strncmp(buffer, "OK", 2) != 0)
    {
        printf("Server rejected store request: %s\n", buffer);
        fclose(fp);
        return;
    }

    printf("Uploading '%s' to server...\n", filename);

    int bytes_read;

    while ((bytes_read = fread(buffer, 1, BUFFER_SIZE, fp)) > 0)
    {
        send(sock, buffer, bytes_read, 0);
        memset(buffer, 0, BUFFER_SIZE);
    }

    fclose(fp);

    usleep(50000);

    send(sock, "EOF_SIGNAL", 10, 0);

    printf("File '%s' successfully uploaded!\n", filename);
}

void retrieve_file(int sock)
{
    char filename[BUFFER_SIZE];
    char save_name[BUFFER_SIZE + 30];
    char buffer[BUFFER_SIZE];

    printf("Enter filename to retrieve from server (download): ");

    fgets(filename, BUFFER_SIZE, stdin);
    filename[strcspn(filename, "\r\n")] = 0;

    send(sock, "RETRIEVE", 8, 0);

    usleep(10000);

    send(sock, filename, strlen(filename), 0);

    memset(buffer, 0, BUFFER_SIZE);

    int bytes = recv(sock,
                     buffer,
                     BUFFER_SIZE - 1,
                     0);

    if (strncmp(buffer, "OK", 2) != 0)
    {
        printf("Server response: %s\n", buffer);
        return;
    }

    snprintf(save_name,
             sizeof(save_name),
             "downloaded_%s",
             filename);

    FILE *fp = fopen(save_name, "wb");

    if (fp == NULL)
    {
        perror("Error creating local file");
        return;
    }

    printf("Downloading '%s'...\n", filename);

    int bytes_received;

    while ((bytes_received = recv(sock,
                                  buffer,
                                  BUFFER_SIZE,
                                  0)) > 0)
    {
        fwrite(buffer, 1, bytes_received, fp);

        if (bytes_received < BUFFER_SIZE)
            break;

        memset(buffer, 0, BUFFER_SIZE);
    }

    fclose(fp);

    printf("File successfully downloaded!\n");
    printf("Saved as '%s'\n", save_name);

    printf("\n========== DOWNLOADED FILE CONTENT ==========\n");

    fp = fopen(save_name, "rb");

    if (fp == NULL)
    {
        printf("Unable to open downloaded file.\n");
        return;
    }

    int total_bytes = 0;

    while ((bytes_received = fread(buffer, 1, BUFFER_SIZE, fp)) > 0)
    {
        fwrite(buffer, 1, bytes_received, stdout);
        total_bytes += bytes_received;
    }

    fclose(fp);

    if (total_bytes == 0)
    {
        printf("No content in the file.\n");
    }

    printf("\n========== END OF FILE ==========\n");
}

int main()
{
    int sock = 0;
    struct sockaddr_in serv_addr;
    int choice;

    if ((sock = socket(AF_INET, SOCK_STREAM, 0)) < 0)
    {
        printf("Socket creation error\n");
        return -1;
    }

    serv_addr.sin_family = AF_INET;
    serv_addr.sin_port = htons(PORT);

    if (inet_pton(AF_INET,
                  "127.0.0.1",
                  &serv_addr.sin_addr) <= 0)
    {
        printf("Invalid address\n");
        return -1;
    }

    if (connect(sock,
                (struct sockaddr *)&serv_addr,
                sizeof(serv_addr)) < 0)
    {
        printf("Connection Failed!.\n");
        return -1;
    }

    printf("Connected to FTP Server successfully!\n");

    while (1)
    {
        printf("\n================ FTP MENU ================\n");
        printf("1. Store File\n");
        printf("2. Retrieve File\n");
        printf("3. Exit\n");
        printf("Enter choice (1-3): ");

        if (scanf("%d", &choice) != 1)
        {
            while (getchar() != '\n');
            continue;
        }

        getchar();

        if (choice == 1)
        {
            store_file(sock);
        }
        else if (choice == 2)
        {
            retrieve_file(sock);
        }
        else if (choice == 3)
        {
            send(sock, "EXIT", 4, 0);
            printf("Disconnecting from servere!\n");
            break;
        }
        else
        {
            printf("Invalid choice!\n");
        }
    }

    close(sock);

    return 0;
}
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>

#define PORT 8888
#define BUFFER_SIZE 1024

int main() {
    int sock = 0;
    struct sockaddr_in serv_addr;
    char ip_query[BUFFER_SIZE];
    char buffer[BUFFER_SIZE] = {0};

    if ((sock = socket(AF_INET, SOCK_STREAM, 0)) < 0) {
        printf("\nSocket Creation Error\n");
        return -1;
    }

    serv_addr.sin_family = AF_INET;
    serv_addr.sin_port = htons(PORT);

    if (inet_pton(AF_INET, "127.0.0.1", &serv_addr.sin_addr) <= 0) {
        printf("\nInvalid address / Address not supported\n");
        return -1;
    }

    if (connect(sock, (struct sockaddr *)&serv_addr, sizeof(serv_addr)) < 0) {
        printf("\n Connection Failed! Ensure server is running.\n");
        return -1;
    }

    printf("Connected to ARP Server.\n");
    printf("Type 'exit'to disconnect.\n");

    while (1) {
        printf("\nEnter Target IP Address: ");
        if (fgets(ip_query, BUFFER_SIZE, stdin) == NULL) break;

        ip_query[strcspn(ip_query, "\r\n")] = 0;

        if (strlen(ip_query) == 0) continue;

        if (strncmp(ip_query, "exit", 4) == 0 || strncmp(ip_query, "bye", 3) == 0) {
            send(sock, "exit", 4, 0);
            printf("Disconnecting...\n");
            break;
        }

        send(sock, ip_query, strlen(ip_query), 0);

        memset(buffer, 0, BUFFER_SIZE);
        int bytes_received = recv(sock, buffer, BUFFER_SIZE - 1, 0);

        if (bytes_received > 0) {
            printf("[RESULT] ARP Resolution for %s:\n", ip_query);
            printf("  -> MAC Address: %s\n", buffer);
        } else {
            printf("Server connection closed.\n");
            break;
        }
    }

    close(sock);
    return 0;
}
