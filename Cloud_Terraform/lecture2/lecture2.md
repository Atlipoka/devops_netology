1. Ранее мы уже инициализировали терраформ, поэтому процесс создания описывать не буду, начну с того, что касается Терраформа:
 * Сервисный аккуант уже был создан с ролью editor
    * ```
      root@vagrant:/home/vagrant/terraform# yc iam service-account list
      +----------------------+-------+
      |          ID          | NAME  |
      +----------------------+-------+
      | ajeqqv8273vi2m4uf1h5 | maxim |
      +----------------------+-------+
      ```
  * Создал iam ключ для сервисного аккаунта
      * ```
        yc iam key create \
        --service-account-id ajeqqv8273vi2m4uf1h5 \
        --folder-name b1gjjlp1h6jc8jeaclal \
        --output key.json
        ```
   * Создал CLI профиль
      * ```
        root@vagrant:/home/vagrant/terraform# yc config profile list
        default
        terraform ACTIVE
        ```
