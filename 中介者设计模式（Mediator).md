### 中介者设计模式（Mediator)


    中介者设计模式：可以引入中介者对象简化和合理化对象之间的通信。
    使用中介者对象后，无需和其它对象进行通信，只需和中介对象通信。
 
    若是每个对象都对其它对象一无所知，只需要和中介对象交互，说明了正确使用这个设计模式
 
    中介模式的核心是一对协议：一个协议是对自身提供功能， 另一个协议是中介者提供功能
 
 
```
struct Postion {
    var distanceFromRunway: Int
    var height: Int
}

protocol Peer {
    var name:String { get }
    func otherPlanDidChangePostion(postion: Postion) ->Bool
}

protocol Mediator {
    func registerPeer(peer: Peer)
    func unRegisterPeer(peer: Peer)
    func changePosition(peer: Peer, position: Postion) -> Bool
}
```
```
class AirplanMediator: Mediator {
    private var peers: [String: Peer]
    init() {
        peers = [String: Peer]()
    }
    
    func registerPeer(peer: Peer) {
        self.peers[peer.name] = peer
    }
    
    func unRegisterPeer(peer: Peer) {
        self.peers.removeValue(forKey: peer.name)
    }
    
    func changePosition(peer: Peer, position: Postion) -> Bool {
        for storePeer in peers.values {
            if peer.name != storePeer.name  {
                return true
            }
        }
        return false
    }
}

class Airplane: Peer {
    var name: String
    var currentPosition: Postion
    var mediator: Mediator
    
    init(name: String, currentPosition: Postion, mediator: Mediator) {
        self.name = name
        self.currentPosition = currentPosition
        self.mediator = mediator
    }
    
    func otherPlanDidChangePostion(postion: Postion) -> Bool {
        return postion.distanceFromRunway == self.currentPosition.distanceFromRunway &&
        abs(postion.distanceFromRunway - self.currentPosition.distanceFromRunway) < 1000
    }
    
    func changePosition(newPosition: Postion) {
        self.currentPosition = newPosition
        if mediator.changePosition(peer: self, position: self.currentPosition) {
            print("\(name): too close Abort")
            return
        }
        print("\(name): Landed")
    }
    
    func land() {
        self.currentPosition = Postion.init(distanceFromRunway: 0, height: 0)
        mediator.unRegisterPeer(peer: self)
        print("\(name): Landed")
    }
}
```
```
let mediator: Mediator = AirplanMediator.init()
// initial setup
let british = Airplane.init(name: "BA706", currentPosition: Postion.init(distanceFromRunway: 11, height: 21000), mediator: mediator)
// new plane arrives
let americn = Airplane.init(name: "AA101", currentPosition: Postion.init(distanceFromRunway: 12, height: 22000), mediator: mediator)

british.changePosition(newPosition: Postion.init(distanceFromRunway: 8, height: 10000))
british.changePosition(newPosition: Postion.init(distanceFromRunway: 2, height: 2000))
british.changePosition(newPosition: Postion.init(distanceFromRunway: 1, height: 1000))

// new plane arrives
let cathay = Airplane.init(name: "CX200", currentPosition: Postion.init(distanceFromRunway: 13, height: 22000), mediator: mediator)
// plane lands
british.land()
// plane moves too close
cathay.changePosition(newPosition: Postion.init(distanceFromRunway: 12, height: 22000))
```