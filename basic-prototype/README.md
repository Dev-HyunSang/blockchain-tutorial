# Part1. 기본 프로토타입
[blockchain-tutorial/basic-prototype](https://github.com/mingrammer/blockchain-tutorial/tree/master/basic-prototype)를 참고하여서 개발 및 작성하였습니다.  

## 소개
블록체인은 21세기의 가장 혁명적인 기술 중 하나로써 현재는 성숙 단계이며 아직 완전히 실현되지는 않았다. 블록체인은 본질적으로 분산화된 데이터베이스이다. 그러나 블록체인을 독보적으로 만든 건 블록체인이 프라이빗이 아닌 퍼블릭 데이터베이스라는 것이다. **즉, 블록체인을 사용하는 모든 사람들이 합의하에서만 추가될 수 있다. 또한 블록체인을 활용하면 암호화 화폐 및 스마트 컨트랜트를 만드는 것도 가능하다.**

## 블록(Block)
"블록체인"을 구성하는 "블록"부터 살펴보겠다. 블록체인에서 블록이란 가치있는 정보를 저장하는 데이터 구조이다. 한 예로 비트코인의 블록은 모든 암호화 화폐에 있어 필수적인 정보인 트랙잭션 정보를 저장한다. 또한 블록은 블록의 버전, 현재 시간의 타임스탬프 및 이전 블록의 해쉬값 등의 기술적인 정보 또한 포함하고 있다. 이 튜토리얼에서는 블록체인 또는 비트코인의 스펙을 그대로 구현하지 않을거지만 대신 중요한 정보만을 포함하는 간단한 버전의 블록을 구현할 것이다. 구현한 블록의 구조체는 다음과 같다.

```go
type Block struct {
	Timestamp     int64
	Data          []byte
	PrevBlockHash []byte
	Hash          []byte
}
```

**Timestamp**는 현재 시간의 타임스템프(블록 생성 시간), **Data**는 블록에 포함된 실제 가치를 지닌 정보, **PrevBlockHash**는 Hash가 블록 헤더라는 하나의 독립된 데이터 구조를 이루며 트랙잭션(여기서는 **Data**)이 또 다른 독립된 데이터 구조를 갖는다. 우리는 구현을 단순히 하기 위해 이 둘을 합쳤다.

그렇다면 해시는 어떻게 계산할 수 있을까? 해시 계산법은 블로게인에서 아주 중요한 부분이며, 블록체인을 안전하게 만드는 핵심 기능이다. 해시 계산은 계산적ㅇ로 아주 어려운 작업이며, 빠른 컴퓨터에서조차 시간이 다소 걸리는 작업이다 (이게 바로 사람들을 비트코인을 채굴하기 위해 강력한 GPU를 구매하는 이유이다). 이는 의도적인 설계로, 새로운 블록을 추가하기 어렵게 만들어 블록이 추가된 후에 블록의 수정을 방지하기 위함이다. 이 메커니즘에 대한 논의와 구현은 조금 뒤에 살펴보겠다.

지금은 블록을 구성하는 필드들을 하나로 이은 뒤 이어진 문자열에 대해 SHA-256 해시를 계산할 것이다. 다음과 같이 **SetHash** 메서드를 작성한다.

```go
func (b *Block) SetHash() {
	timestamp := []byte(strconv.FormatInt(b.Timestamp, 10))
	headers := bytes.Join([][]byte{b.PrevBlockHash, b.Data, timestamp}, []byte{})
	hash := sha256.Sum256(headers)
	b.Hash = hash[:]
}
```

다음은 Go의 컨벤션을 따라, 블록을 생성하는 함수를 작성한다.

```go
func NewBlock(data string, prevBlockHash []byte) *Block {
	block := &Block{
		Timestamp:     time.Now().Unix(),
		Data:          []byte(data),
		PrevBlockHash: prevBlockHash,
		Hash:          []byte{},
	}
	block.SetHash()
	return block
}
```
블록을 위한 코드 작성은 끝났다!

## 블록체인(Blockchanin)
이제 블록체인을 구현해보자. 본질적으로 블록체인은 특정한 구조를 가진 데이터베이스일 뿐이며 순서가 지정된 링크드 리스트이다. 즉, 블록은 삽입 순서대로 저장되며 각 블록은 이전 블록과 연결된다. 이러한 구조 덕분에 최신 블록을 빠르게 가쳐올 수 있고 해시를 블록을 (효율적으로) 검색할 수 있다.  

Go에서는 배열과 맵을 활용해 이 구조를 구현할 수 있다. 배열은 정렬된 해시를 유지하고 맵은 해시-블록쌍을 유지한다. 그러나 지금 당장은 해시 검색 기능이 필요하지 않기 때문에 프로토타입 구현에서는 배열만 사용할 것이다.

```go
type Blockchain struct {
	blocks []*Block
}
```
우리의 첫 블록체인이 완성되었다. 아주 쉽다.

블록 추가 기능을 만들어 보겠다.
```go
func (bc *Blockchain) AddBlock(data string) {
	prevBlock := bc.blocks[len(bc.blocks)-1]
	newBlock := NewBlock(data, prevBlock.Hash)
	bc.blocks = append(bc.blocks, newBlock)
}
```
새 블록을 추가하려면 기존 블록이 필요하다. 그러나 우리 블록체인에는 아무 블록도 없다. 따라서 모든 블록체인에는 적어도 하나의 블록이 있어야하며, 블록체인의 첫번째 블록을 **제네시스 블록**이라고 한다. 이 블록을 생성하는 메서드를 구현해보자.
```go
func NewGenesisBlock() *Block {
	return NewBlock("Genesis Block", []byte{})
}
```
이제 제네시스 블록을 가지고 블록체인을 생성하는 함수를 구현해보자.
```go
func NewBlockchain() *Blockchain {
	return &Blockchain{[]*Block{NewGenesisBlock()}}
}
```
제대로 동작하는지 확인해 보겠다.
```go
func main() {
	bc := NewBlockchain()

	bc.AddBlock("Send 1 BTC to Ivan")
	bc.AddBlock("Send 2 more BTC to Ivan")

	for _, block := range bc.blocks {
		fmt.Printf("Prev. hash: %x\n", block.PrevBlockHash)
		fmt.Printf("Data: %s\n", block.Data)
		fmt.Printf("Hash: %x\n", block.Hash)
		fmt.Println()
	}
}
```
**출력:**
```shell
Prev. hash: 
Data: Genesis Block
Hash: 4c8225502c1656d0561f120ed0e0ae9bad5487a631cae7b303e8e29900dcfd84

Prev. hash: 4c8225502c1656d0561f120ed0e0ae9bad5487a631cae7b303e8e29900dcfd84
Data: Send 1 BTC to Ivan
Hash: 65d7c77c3a2bdc5097d5031a86f38e75bdd26016d53df2beae68d64ec1720034

Prev. hash: 65d7c77c3a2bdc5097d5031a86f38e75bdd26016d53df2beae68d64ec1720034
Data: Send 2 more BTC to Ivan
Hash: 6188f6ff1086929b50fa293a6af745f0ccec1753dd9616b7de1d8b227e862966
```

## 결론
이제까지 매우 간단한 블록체인 프로토타입을 만들어보았다. 이는 단지 블록들의 배열로 각 블록은 이전 블록과 연결되어있다. 실제 블록체인은 이보다 훨씬 더 복잡하다. 우리가 만든 블록체인에서 새로운 블록을 추가하는 것은 쉽고 빠르지만 실제 블록체인에서 새로운 블록을 추가하는데는 몇 가지 작업이 필요하다: 블록 추가 권한을 얻기 위해 아주 무거운 계산을 수행해야한다 (이 메커니즘을 작업 증명 (Proof of Work)이라고 한다). 또한 블록체인은 단일 의사결정자가 없는 분산화된 데이터베이스이다. 따라서, 하나의 새로운 블록은 반드시 네트워크의 참여자들로부터 확인과 승인을 받아야한다. (이 메커니즘을 합의 또는 컨센서스 (consensus)라고 한다)). 그리고 우리의 블록체인에는 아직 트랜잭션이 없다!