## Nadai Starknet UDC con Protostar y L2EsERC721

En este tutorial aprenderemos algunos de los nuevos cambios que se están añadiendo en Starknet. Utilizaremos el contrato de implementación universal [UDC], en este tutorial conseguiremos el `Class Hass` desde `Protostar`. 

Usaremos varias herramientas para hacer un `deploy` de un contrato ERC721, este código de metadata contiene imágenes que usaremos con la información para `Mint` los `7 POAP` conseguidos en los Workshop [L2 Español](https://t.me/s/l2espaniol) y [Starknetes](https://t.me/s/starknet_es).  Puede revisarlos toda la [PLAYLIST aquí](https://www.youtube.com/playlist?list=PL5LoUunXvIgLCdVerVBPZ2G3bR51Re251)

Herramientas o link directos.

* Instalación [Protostar](https://github.com/software-mansion/protostar)
* Instalación [Starkscan](https://github.com/starkscan/starkscan-verifier)
* Utils Stark [Convertidor a Felt](https://www.stark-utils.xyz/converter)
* Wallet [Braavos](https://braavos.app/)
* Wallet [ArgentX](https://www.argent.xyz/argent-x/)
* Starknet L2 [Faucet](https://faucet.goerli.starknet.io/)
* Goerli L1 [Faucet Goerli](https://goerlifaucet.com/)
* Goerli L1 [Faucet Chain](https://faucets.chain.link/)
* Goerli L1 [Faucet Paradigm](https://faucet.paradigm.xyz/)
* Bridge Oficial [Starkgate](https://goerli.starkgate.starknet.io/)
* Exploradores de Bloques Tesnet [Starkscan](https://testnet.starkscan.co/)
* Exploradores de Bloques Tesnet [Voyager](https://goerli.voyager.online/)
* Exploradores de Bloques Tesnet2 [Starkscan](https://testnet-2.starkscan.co/)
* Exploradores de Bloques Tesnet2 [Voyager](https://goerli-2.voyager.online/)
* Guia enviar de [Goerli L1 a Testnet2 L2 Starknet](https://github.com/Nadai2010/Nadai-Testnet-2-Starknet)


* Como alternativa, también podrá usar directamente la [CLI Starknet](https://medium.com/starknet-edu/deploying-to-starknet-with-the-universal-deployer-contract-c6de07092bfb) siguiendo este tutorial.


**NINGUNO DE ESTOS POAP SON ORIGINALES Y SOLO SERÁN PARA EL DISFRUTE VISUAL DADO QUE NO TIENEN MÁS VALOR QUE LA EXPERIENCIA DEL APRENDIZAJE.**

---

### El Contrato de Implementación Universal (UDC)

La gente de OpenZeppelin ha sido consciente de la necesidad de tener un contrato inteligente de implementación que sea lo suficientemente genérico como para que pueda ser utilizado por cualquiera que desee implementar sus contratos inteligentes. Lo han llamado Universal Deployer Contract (UDC), que tiene una sola función, `deploymentContract`.

Para implementar su contrato utilizando el UDC, ahora debemos realizar varios pasos.

1. Compile su contrato.
2. Declare su clase de contrato en su red de destino (Goerli o Mainnet) y tome nota del `Class Hass` devuelto.
3. Invoque la función `deploymentContract` del UDC en Goerli o Mainnet, pasando el `Class Hass` de su contrato y los argumentos del constructor, entre otros parámetros.

---

## Protostar

Una vez tengamos instalado Protostar, empezaremos un nuevo proyecto con el siguiente comando.

```bash
protostar init
```

![Graph](/im%C3%A1genes/init.png)

Esto nos dará los ajustes básicos y el archivo [protostar.toml](/protostar.toml), en el que le indicaremos el nombre y ruta del contrato. 

![Graph](/im%C3%A1genes/toml.png)

Así que ahora copie el [L2EsERC721.cairo](/src/L2EsERC721.cairo) y péguelo en su contrato creado de `main.cairo` y cambiele el nombre. También puede directamente crear un nuevo archivo, poner el nombre que quiera y pegar el código.

En nuestro caso instalamos las librerias de [cairo-contracts](/cairo-contracts/), usted podra instalar o clonar usando los siguiente comando, aunque si clonais directamente esta REPO no les haría falta.

```bash
gh repo clone OpenZeppelin/cairo-contracts
```

O también podria usar

```bash
pip install openzeppelin-cairo-contracts
```

![Graph](/im%C3%A1genes/oppen.png)

---

## Protostar Compile y Class Hass

Ahora primero probaremos a Compilar nuestro contrato, tenemos que fijarnos que todas las librerias, utils y contratos estén en el proyecto. Si tiene alguna duda puede revisar el Contrato [L2EsERC721.cairo](/src/L2EsERC721.cairo) los `From` para saber bien que se le está pidiendo. Aunque solo clonando la repo no deberia tener ningún problema. El contrato que vamos a compilar lo hemos indicado en el archivo [protostar.toml](/protostar.toml), asi que usaremos los comandos.

```bash 
protostar build
```

Ahora si todo ha ido bien y nos ha dado en la carpeta [build](/build/L2EsERC721.json) los dos archivos `.json` pues todo ha ido bien. 

![Graph](/im%C3%A1genes/build1.png)
![Graph](/im%C3%A1genes/build.png)

Ahora toca delarar el contrato en la red que vayamos usar, en nuestro caso en `Testnet` y nos dará el `Class Hass` que nos hará falta para hacer el `deploymentContract` del UDC. Para eso usamos el siguiente comando.

```bash
protostar declare build/L2EsERC721.json --network testnet
```

![Graph](/im%C3%A1genes/declare.png)

* Aqui obtenemos Class hash: `0x07979ccad72fc1cf8eb8d03880c303e9c0d0f8908941203c8b1490844a80c79b`
* Aquí obtemos como se ha enviado la transacción del Class Hass en la [Testnet](https://testnet.starkscan.co/class/0x07979ccad72fc1cf8eb8d03880c303e9c0d0f8908941203c8b1490844a80c79b)

---

## Preparando Argumentos del constructor para el Deploy.

En este código necesitamos pasar varios argumentos a `felt` si nos fijamos en el constructor necesitamos:

1. name: felt (Nombre que queramos poner al Contrato)
2. symbol: felt (Simbolo que llevara nuestro Contrato)
3. owner: felt (Owner del Contrato)
4. base_token_uri_len: felt (Largo de la base del token_uri, para esta repo `3`)
5. base_token_uri: felt* (Pondremos por separado con `,` y máximo 31 caracteres pasados a felt de la metada, para esta repo no necesitará hacer nada)
6. token_uri_suffix: felt (Sufijo en el que terminará su metadata, en esta repo `.json`)

* Utils Stark [Convertidor a Felt](https://www.stark-utils.xyz/converter)

Nuestra conversión nos ha dado.

1. 104185472425128416901810772103455519281 // Nadai L2EsERC721
2. 5129801 // NFI
3. 1795950254530259382270168937734171348535331377400385313842303804539016002736 // 0x03F878C94De81906ba1A016aB0E228D361753536681a776ddA29674FfeBB3CB0
4. 3
5. 184555836509371486644298270517380613565396767415278678887948391494588524912,  181013377130037319514477958885842869644306949936208462993653051087166982263, 2326625651510011578704269826301011662181198383 // https://gateway.pinata.cloud/ip, fs/QmRHH2skwE8ScAXwwmjnX8XKViHw, hTZFJBaavq3KHsCdWj/
6. 199354445678 // .json

Ahora que tenemos todos convertidos y tenemos nuestro `Class Hass`, estamos listo para implementar el `Deploy` de nuestro ERC721, pero primero aprendamos un poco más sobre algunas definiciones antes de pasar a la implementación real a través del contrato de implementación universal. La dirección UDC es ]la misma en todas las redes, puede consultarla [aquí](https://testnet.starkscan.co/contract/0x041a78e741e5af2fec34b695679bc6891742439f7afb8484ecd7766661ad02bf#write-contract)]. Como puede ver, tiene un único método externo, `deploymentContract`, que espera: 

* ClassHash: La clase del contrato que desea implementar, que hemos calculado y declarado previamente.
* Salt: El valor de la `salt` puede ser cualquier número que desee; está ahí solo para introducir aleatoriedad en la dirección generada para su contrato inteligente que pronto se implementará. 
* Unique: El campo único , combinado con salt , se puede usar para obtener la misma dirección en diferentes redes. Por ejemplo, si implemento en Goerli, pasando el valor `5 para salt` y `0 (Falso) para único`, mi contrato inteligente se implementará en una dirección específica; digamos que es `0xabc...` Ahora puedo repetir el proceso de implementación, pero esta vez en Mainnet, y si paso los mismos valores para `salt y unique (5 y 0)`, mi contrato inteligente se implementará en la misma dirección `0xabc...` pero esto tiempo en Mainnet. En mi ejemplo, usaré el valor 0 para ambos parámetros por simplicidad, no porque quiera conservar la dirección.
* Calldata_len: Cantidad de argumentos necesarios para el constructor, en nuestro caso son 6 pero la base_uri se compone de 3 argumentos, así que contando que en 6 ya teniamos una sumada serián 8 los argumentos de constructor necesarios para el deploy.
* Calldata: Los argumentos del constructor pasados a `felt` en nuestro caso los 8 calculados antes. 

NOTA: Tenga en cuenta que para su `deploy` necesitará poner su `owner` en el `3 paso` al convertirlo a `felt`.

---

## Deploy con UDC 

Iremos al contrato preparado para el [DeploymentContract](https://testnet.starkscan.co/contract/0x041a78e741e5af2fec34b695679bc6891742439f7afb8484ecd7766661ad02bf#write-contract) y pasaremos los argumentos. 

![Graph](/im%C3%A1genes/UDC.png)

* [HASH DeploymentContract L2EsERC721](https://testnet.starkscan.co/tx/0x145969fab600e7ef0c6d779b44d72557431583a2ef15ffb1175bdda8b4576d4)

Si todo ha ido bien nuestro L2EsERC721 estará deployado, ahora usaremos la herramienta [Starkscan](https://github.com/Nadai2010/Nadai-Starkscan-Verify) para verificar nuestro contrato de una forma rápida. Una vez instalado, dentro del directorio del proyecto con el comando:

```bash
npm install -g starkscan
```

Podrá ejecutar en su terminal, le saldrá una lista con todos los smart de cairo disponible, en nuestro caso escogeremos `L2EsERC721.cairo`. Y tendremos que añadir nuestro `ClassHass` o `Contract Address`, indicar la Testnet y al terminar su contrato ya estará verficado.

```bash
starkscan
```

![Graph](/im%C3%A1genes/stark.png)

* [Contract Verify](https://testnet.starkscan.co/class/0x07979ccad72fc1cf8eb8d03880c303e9c0d0f8908941203c8b1490844a80c79b#code)

---

## Mint de POAP NFT L2 Español y Starknetes en Workshop

Ahora si ya ha ido todo bien puede ir a su nuevo contrato desplegado y `Mint` las imágenes de los `POAP` de los Workshop. Deberá de mintear `7 VECES` para conseguir uno de cada modelo, a partir del `8 NO TENDRÁ IMAGEN`. Una vez minteado podra ir a [MINTSQUARE](https://mintsquare.io/starknet-testnet) y ver su colección.

Para ello deberemos ir en nuestro caso [L2EsStarknetes](https://testnet.starkscan.co/contract/0x03a1c616b57ce3e9f46367441bf41aef51a3add98b771a0cd01654b009875a7e#write-contract) en el paso `5 mint` y añadimos `to` y el `Id` que vamos a crear. En mi caso ha sido mi wallet y el `Id` primero el 1, luego 2,...hasta el 7.

![Graph](/im%C3%A1genes/mint.png) ![Graph](/im%C3%A1genes/mint2.png)


Asi podrá ver su colleción después de Mintear toda su colección

![Graph](/im%C3%A1genes/colec.png)

---

## Conclusión Final

Implementar contratos inteligentes de StarkNet utilizando el `UDC` de OpenZeppelin. El UDC utiliza la nueva llamada al sistema de implementación para que el Secuenciador pueda cobrar cuando implementa contratos inteligentes, un requisito para descentralizar la red. El uso de una llamada al sistema también nos permite integrar implementaciones en la ejecución de cualquier contrato inteligente que creemos.

Otra forma de declarar e implementar un contrato inteligente con UDC es mediante el uso de las herramientas de desarrollo de Argent X disponibles en la extensión de su navegador. Puede usar sus herramientas abriendo la extensión de su navegador y luego yendo a `Configuración → Configuración del desarrollador → Desarrollo de contratos inteligentes` Allí podrá tanto declarar como deployar su contrato pasando los datos requeridos, como `ClassHass` o argumentos. Esto es probablemente más fácil que usar la CLI y tiene el beneficio adicional de rastrear el saldo de su Ether de prueba.

