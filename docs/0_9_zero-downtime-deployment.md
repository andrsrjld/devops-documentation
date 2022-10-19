# Zero Downtime Docker Container (*with Docker Swarm)

Pertanyaan. Bagaimana bisa kita melakukan update terhadap Docker Container yang kita jalankan  tidak akan mematikan container yang sedang berjalan untuk melakukan update versi aplikasi yang dijalankan di container tersebut?

Sebelumnya hal yang saya pelajari di dalam dunia per-"docker"-an, ketika anda ingin melakukan update terhadap image yang dijalankan akan mematikan container yang sedang berjalan dan akan mengganti dengan container dan image yang baru. Bagaimana kita bisa melewati langkah "mematikan" container yang sedang berjalan tersebut, sehingga kita memiliki aplikasi dengan ~99% availibility.

Hal yang saya temukan setelah membaca dan menonton banyak sekali sumber di internet ada dua aplikasi yang dapat membantu hal ini: Kubernetes dan Docker Swarm. Terlihat bahwa Kubernetes memang sudah dirancang dari awal untuk melakukan orkestrasi container sehingga "zero downtime" merupakan fitur utama dalam aplikasi itu. Tetapi, Docker Swarm merupakan ekstensi dari aplikasi Docker yang memiliki beberapa fitur serupa dengan Kubernetes. Disini saya melakukan eksperimen dengan Docker Swarm dikarenakan ada banyak hal yang perlu dipelajari kembali jika kita memilih Kubernetes.

### Start Learning on Docker Swarm and Rolling Update

Pertama, siapkan Docker Swarm sesuai dengan tutorial dokumentasi [Create a swarm | Docker Documentation](https://docs.docker.com/engine/swarm/swarm-tutorial/create-swarm/). Setelah disiapkan Docker Swarm-nya, yang harus dipikirkan selanjutnya merupakan bagaimana kita dapat mendeploy aplikasi di Docker Swarm. Untuk membaut gambaran apa saja yang akan di deploy kita bisa menggunakan "Docker Compose File." File ini digunakan untuk mendaftarkan apa saja properti yang akan dijalankan oleh Docker Swarm. Contoh file yang dibuat seperti di bawah:

    # How to "balance services" with multiple container
    # https://stackoverflow.com/questions/47838031/ several-replicas-into-docker-swarm

    version: "3.9"

    services:
      web:
        image: gcr.io/bpp-rts-prod/agent-app-api:Production
        ports:
          - "8080:8080"
        networks:
          - agent-app-api
        deploy:
          replicas: 4

    networks:
      agent-app-api:
        driver: overlay

File docker-compose.yaml yang dibuat memberitahu Docker untuk membuat "service" yang dinamakan web, diambil dari image kaenova/ci-cd-playground:latest. Melakukan port forwarding dari host 2001 ke container 2000 yang aritnya kita akan masuk ke dalam aplikasi melalui port 2001 dan akan diarahkan oleh docker ke port 2000. Dan membuat 2 container dari serivce yang dibuat.

Hal yang penting disini ialah tulisan latest pada image yang akan kita ambil. Selain itu kita harus membuat overlay networks untuk melakukan "load balancing" dari container yang kita buat dan memastikan service kita menggunakan networks yang digunakan.

Bagaimana kita jalankan aplikasi yang sudah disiapkan ini? Kita bisa menjalankan command di bawah ini:

    $ docker stack deploy -c docker-compose.yaml --prune <Nama Stack / Aplikasi>

Disini -c digunakan untuk memberitahu file compose kita, dan --prune digunakan untuk menghapus service yang tidak digunakan. Command ini juga bisa digunakan terhadap aplikasi yang sudah berjalan dan akan memperbarui berdasarkan pendeteksian image yang baru.

Agar lebih baik kita bisa menambahkan command di bawah untuk menghapus semua container dan image yang sudah tidak digunakan.

    docker container prune && docker image prune

### CI / CD?

Oke karena kita sudah tahu bagaimana bisa melakukan balancing, dan rolling update terhadap aplikasi yang kita siapkan, bagaimana kita bisa melakukan update berdsarkan pembaharuan kode yang ada?

Disini kita bisa menggunakan Gitlab Pipeline untuk membuat image - upload ke Gitlab Conainer Registry - jalankan command update di server.

Tapi jangan lupa bahwa kita masih harus menyiapkan Docker Swarm dan file Docker Composenya untuk melakukan semua itu. 

    stages:
      - Build
      - Deploy

    Kaniko Build:
      stage: Build
      image:
        name: gcr.io/kaniko-project/executor:debug
        entrypoint: [""]
      tags:
        - gke-devops-ptpr
      only:
        - Production
      script:
        - echo $GCLOUD_SERVICE_KEY_PROD > $CI_PROJECT_DIR/service_account_key.json
        - export GOOGLE_APPLICATION_CREDENTIALS=$CI_PROJECT_DIR/service_account_key.json
        - >-
          /kaniko/executor
          --context=$CI_PROJECT_DIR
          --dockerfile=$CI_PROJECT_DIR/Dockerfile
          --destination=gcr.io/bpp-rts-prod/agent-app-api:Production

    Deploy Production:
      stage: Deploy
      image: alpine:latest
      tags: 
        - gke-devops-ptpr
      only:
        - Production
      before_script:
        - apk add --update --no-cache openssh sshpass net-tools
        - df
        - cat /etc/resolv.conf
        - cat /etc/hosts
        - netstat -tulpn
      script:
        - sshpass -p $SERVER_PASS_PROD ssh -tt -oStrictHostKeyChecking=no $SSH_USER@$SERVER_IP_PROD 'docker pull gcr.io/bpp-rts-prod/agent-app-api:Production'
        - sshpass -p $SERVER_PASS_PROD ssh -tt -oStrictHostKeyChecking=no $SSH_USER@$SERVER_IP_PROD 'cd /home/rts_admin/zero-downtime && docker service update --force agent-app-api_app'