lab4
====

TCP Client
====================
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <time.h>

int main(int argc, const char *argv[])
{
  int sockfd;
	char buffer[256];
	int bytes;
	struct sockaddr_in sa;

	sa.sin_family = AF_INET;
	sa.sin_port = htons(6666);
	/*sa.sin_addr.s_addr = htonl((128<<24) | 1);*/
	inet_aton("127.0.0.1", &sa.sin_addr);

	if((sockfd=socket(PF_INET,SOCK_STREAM,0)) < 0){
		perror("socket error");
		exit(EXIT_FAILURE);
	};
	printf("Socket ok\n");
	struct timeval timeout;      
	timeout.tv_sec = 5;
	timeout.tv_usec = 0;

	if (setsockopt (sockfd, SOL_SOCKET, SO_RCVTIMEO, (char *)&timeout,
				sizeof(timeout)) < 0)
		error("setsockopt failed\n");

	if (setsockopt (sockfd, SOL_SOCKET, SO_SNDTIMEO, (char *)&timeout,
				sizeof(timeout)) < 0)
		error("setsockopt failed\n");

	if(connect(sockfd,(struct sockaddr*)&sa,sizeof sa) < 0){
		perror("connect error");
		close(sockfd);
		exit(EXIT_FAILURE);
	}
	printf("Connection ok\n");
	printf("Initialization complete\n");

	while((bytes = read(sockfd,buffer,256)) > 0)
		printf("%s\n", buffer);

	close(sockfd);
	return 0;
}
===============================

TCP Server
================================
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>

int main(int argc, const char *argv[])
{
  int sockfd;
	struct sockaddr_in name;

	FILE *client;

	if((sockfd= socket(PF_INET,SOCK_STREAM,0)) == -1){
		perror("socket error");
		exit(EXIT_FAILURE);
	};

	name.sin_family = AF_INET;
	name.sin_port = htons(6666);
	name.sin_addr.s_addr = htonl(INADDR_ANY);

	if (bind (sockfd, (struct sockaddr *) &name, sizeof (name)) < 0)
	{
		perror ("bind");
		exit (EXIT_FAILURE);
	}

	listen(sockfd,1);

	switch(fork()){
		case -1:
			perror("fork");
			exit(EXIT_FAILURE);
		case 0:
			close(sockfd);
			exit(0);
		default:
			printf("Listening\n");
			break;
	}

	int c;
	for(;;){
		int b = sizeof(name);
		if((c = accept(sockfd,(struct sockaddr*)&name, &b)) < 0){
			perror("Accept");
			return 4;
		}
		if((client = fdopen(c,"w")) == NULL){
			perror("fdopen");
			return 5;
		}
		printf("Wow,a client\n");
		break;
	}

	fprintf(client, "Hello, client!");
	fclose(client);
	shutdown(sockfd,2);
	close(sockfd);
	return 0;
}
========================================

UDP Client
========================================
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <time.h>

int main(int argc, const char *argv[])
{
  int sockfd;
	char buffer[256];
	int bytes;
	struct sockaddr_in sa;

	sa.sin_family = AF_INET;
	sa.sin_port = htons(6666);
	inet_aton("127.0.0.1", &sa.sin_addr);

	if((sockfd=socket(PF_INET,SOCK_DGRAM,0))<0){
		perror("socket error");
		exit(EXIT_FAILURE);
	};
	struct timeval timeout;      
	timeout.tv_sec = 5;
	timeout.tv_usec = 0;

	const char msg[256] = "Pew!\n";
	for(;;){
		if(sendto(sockfd,msg,256,MSG_NOSIGNAL,(struct sockaddr*)&sa,sizeof sa) < 0){
			perror("sendto");
			close(sockfd);
			exit(EXIT_FAILURE);
		}
	}
	shutdown(sockfd,2);

	close(sockfd);
	return 0;
}

========================================
UDP Server
========================================
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>

int main(int argc, const char *argv[])
{
  int sockfd;
	struct sockaddr_in name;

	FILE *client;

	if((sockfd= socket(PF_INET,SOCK_DGRAM,0)) < 0){
		perror("socket error");
		exit(EXIT_FAILURE);
	};

	name.sin_family = AF_INET;
	name.sin_port = htons(6666);
	name.sin_addr.s_addr = htonl(INADDR_ANY);

	if (bind (sockfd, (struct sockaddr *) &name, sizeof (name)) < 0)
	{
		perror ("bind");
		exit (EXIT_FAILURE);
	}

	int bytes;
	char rmsg[256];
	while(1){
		if((bytes = recvfrom(sockfd,rmsg,256,MSG_NOSIGNAL,(struct sockaddr*)NULL,NULL)) < 0){
			perror("recvfrom");
			close(sockfd);
			exit(EXIT_FAILURE);
		}
		printf("Message: %s\n", rmsg);
	}

	shutdown(sockfd,2);
	close(sockfd);
	return 0;
}
