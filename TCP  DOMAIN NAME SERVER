#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <stdint.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/types.h>
#include <sys/wait.h>

#define BUFFER_SIZE 1024
#define TABLE_SIZE 10

typedef struct Node
{
    char domain[100];
    char ip[50];
    struct Node *next;
} Node;

Node *hashTable[TABLE_SIZE];

int hashFunction(char *domain)
{
    int hash = 0;

    for (int i = 0; domain[i] != '\0'; i++)
    {
        hash = (hash + domain[i]) % TABLE_SIZE;
    }

    return hash;
}

void insert(char *domain, char *ip)
{
    int index = hashFunction(domain);

    Node *newNode = (Node *)malloc(sizeof(Node));

    strcpy(newNode->domain, domain);
    strcpy(newNode->ip, ip);

    newNode->next = hashTable[index];
    hashTable[index] = newNode;
}

char *search(char *domain)
{
    int index = hashFunction(domain);

    Node *current = hashTable[index];

    while (current != NULL)
    {
        if (strcmp(current->domain, domain) == 0)
        {
            return current->ip;
        }

        current = current->next;
    }

    return NULL;
}

int main(int argc, char *argv[])
{
    int sockfd;
    int port;

    struct sockaddr_in server_addr;
    struct sockaddr_in client_addr;

    char domain[BUFFER_SIZE];
    char response[BUFFER_SIZE + 128];

    for (int i = 0; i < TABLE_SIZE; i++)
    {
        hashTable[i] = NULL;
    }

    insert("google.com", "142.250.195.14");
    insert("youtube.com", "142.250.72.206");
    insert("facebook.com", "157.240.241.35");
    insert("example.com", "93.184.216.34");
    insert("amazon.com", "98.137.11.163");
    insert("github.com", "140.82.114.4");
    insert("wikipedia.org", "208.80.154.224");
    insert("instagram.com", "157.240.241.174");
    insert("microsoft.com", "20.112.250.133");
    insert("apple.com", "17.253.144.10");

    if (argc != 2)
    {
        printf("Usage: %s <port>\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    port = atoi(argv[1]);

    if (port <= 0 || port > 65535)
    {
        printf("Invalid port number\n");
        exit(EXIT_FAILURE);
    }

    sockfd = socket(AF_INET, SOCK_DGRAM, 0);

    if (sockfd < 0)
    {
        perror("Socket creation failed");
        exit(EXIT_FAILURE);
    }

    memset(&server_addr, 0, sizeof(server_addr));

    server_addr.sin_family = AF_INET;
    server_addr.sin_port = htons((uint16_t)port);
    server_addr.sin_addr.s_addr = INADDR_ANY;

    if (bind(sockfd,
             (struct sockaddr *)&server_addr,
             sizeof(server_addr)) < 0)
    {
        perror("Bind failed");
        close(sockfd);
        exit(EXIT_FAILURE);
    }

    printf("DNS UDP Concurrent Server started on port %d\n", port);
    printf("Hash table size: %d\n", TABLE_SIZE);
    printf("Server is ready for multiple clients.\n\n");

    while (1)
    {
        socklen_t addr_len = sizeof(client_addr);

        ssize_t bytes_received = recvfrom(
            sockfd,
            domain,
            BUFFER_SIZE - 1,
            0,
            (struct sockaddr *)&client_addr,
            &addr_len
        );

        if (bytes_received < 0)
        {
            perror("recvfrom failed");
            continue;
        }

        domain[bytes_received] = '\0';

        printf("Client [%s:%d] requested: %s\n",
               inet_ntoa(client_addr.sin_addr),
               ntohs(client_addr.sin_port),
               domain);

        pid_t pid = fork();

        if (pid < 0)
        {
            perror("fork failed");
            continue;
        }

        if (pid == 0)
        {
            char *ip = search(domain);

            if (ip != NULL)
            {
                snprintf(response,
                         sizeof(response),
                         "Domain: %s\nIP Address: %s",
                         domain,
                         ip);
            }
            else
            {
                snprintf(response,
                         sizeof(response),
                         "Domain not found");
            }

            printf("Child [%d] Result: %s\n\n",
                   getpid(), response);

            if (sendto(sockfd,
                       response,
                       strlen(response),
                       0,
                       (struct sockaddr *)&client_addr,
                       addr_len) < 0)
            {
                perror("sendto failed");
            }

            close(sockfd);
            exit(0);
        }

        else
        {
            printf("Parent: Created child process %d\n\n", pid);

            waitpid(-1, NULL, WNOHANG);
        }
    }

    close(sockfd);

    return 0;
}

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <stdint.h>
#include <unistd.h>
#include <arpa/inet.h>

#define BUFFER_SIZE 1024

int main(int argc, char *argv[])
{
    int sockfd;
    int port;

    struct sockaddr_in server_addr;

    char domain[BUFFER_SIZE];
    char response[BUFFER_SIZE];

    if (argc != 3)
    {
        printf("Usage: %s <server_ip> <server_port>\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    port = atoi(argv[2]);

    if (port <= 0 || port > 65535)
    {
        printf("Invalid port number\n");
        exit(EXIT_FAILURE);
    }

    sockfd = socket(AF_INET, SOCK_DGRAM, 0);

    if (sockfd < 0)
    {
        perror("Socket creation failed");
        exit(EXIT_FAILURE);
    }

    memset(&server_addr, 0, sizeof(server_addr));

    server_addr.sin_family = AF_INET;
    server_addr.sin_port = htons((uint16_t)port);

    if (inet_pton(AF_INET,
                  argv[1],
                  &server_addr.sin_addr) <= 0)
    {
        printf("Invalid server IP address\n");
        close(sockfd);
        exit(EXIT_FAILURE);
    }

    socklen_t addr_len = sizeof(server_addr);

    printf("Connected to DNS Server %s:%d\n",
           argv[1], port);

    printf("Type 'exit' to stop.\n\n");

    while (1)
    {
        printf("Enter Domain Name: ");

        if (fgets(domain, BUFFER_SIZE, stdin) == NULL)
        {
            break;
        }

        domain[strcspn(domain, "\n")] = '\0';

        if (strcmp(domain, "exit") == 0)
        {
            printf("DNS client terminated.\n");
            break;
        }

        if (sendto(sockfd,
                   domain,
                   strlen(domain),
                   0,
                   (struct sockaddr *)&server_addr,
                   addr_len) < 0)
        {
            perror("sendto failed");
            break;
        }

        ssize_t bytes_received = recvfrom(
            sockfd,
            response,
            BUFFER_SIZE - 1,
            0,
            NULL,
            NULL
        );

        if (bytes_received < 0)
        {
            perror("recvfrom failed");
            break;
        }

        response[bytes_received] = '\0';

        printf("\nDNS Server Response:\n");
        printf("%s\n\n", response);
    }

    close(sockfd);

    return 0;
}
