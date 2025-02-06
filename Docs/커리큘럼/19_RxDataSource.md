# RxDataSources

이것은 `UITableView`와 `UICollectionView`를 위한 완전한 기능의 반응형 데이터소스를 구현한 클래스 집합니다.

RxDataSource are bundled [here.](https://github.com/RxSwiftCommunity/RxDataSources)  
여기에 함께 포함되어있습니다.

Fully functional demonstration how to use them is included in the [RxExample](https://github.com/ReactiveX/RxSwift/tree/main/RxExample) project.  
.RxExample 프로젝트에는 이를 사용하는 완전한 데모가 포함되어 있습니다.

일반적인 데이터소스 (`delegate`, `dataSource패턴`)를 사용하는 대신 ,`Observable` 스트림을 통해 UI와 데이터를 자동으로 연결합니다.

특징으로는  
데이터 변화에 따라  `tableView`와 `collectionView`에 자동으로 업데이트 됨 
섹션 단위의 데이터 관리가 용이함 .

> 사용예제

> 섹션 모델 정의
```swift
struct SectionModel<ItemType>: SectionModelType {
    var header: String
    var items: [ItemType]

    init(header: String, items: [ItemType]) {
        self.header = header
        self.items = items
    }

    init(original: SectionModel<ItemType>, items: [ItemType]) {
        self = original
        self.items = items
    }
}
```

> 적용하기
```swift
import UIKit
import RxSwift
import RxCocoa
import RxDataSources

class ViewController: UIViewController {
    let disposeBag = DisposeBag()
	let tableView = UITableView()
	var dataSource: RxTableViewSectionedReloadDataSource<SectionModel<String>>!


// 초기 데이터

    let initialSections = [
     SectionModel(header: "Fruits", items: ["Apple", "Banana", "Orange"]),
     SectionModel(header: "Vegetables", items: ["Carrot", "Tomato", "Broccoli"])      ]

// 업데이트될 데이터
    let updatedSections = [
        SectionModel(header: "Fruits", items: ["Mango", "Pineapple"]),
        SectionModel(header: "Vegetables", items: ["Spinach", "Pumpkin"])
    ]


    override func viewDidLoad() {
        super.viewDidLoad()
        setupUI()
        setupTableView()
        bindTableView()
    }
    
    func setupUI() {
// 버튼,특성 뷰의색상, 레이아웃 추가, 레이아웃 설정 생략 (기본 설정생략) 
        // 테이블뷰 등록
        tableView.register(UITableViewCell.self, forCellReuseIdentifier: "Cell")
    }
    
    // 테이블뷰 설정
    func setupTableView() {
        dataSource = RxTableViewSectionedReloadDataSource<SectionModel<String>>(
            configureCell: { _, tableView, indexPath, item in
                let cell = tableView.dequeueReusableCell(withIdentifier: "Cell", for: indexPath)
                cell.textLabel?.text = item
                return cell
            },
            titleForHeaderInSection: { dataSource, index in
                return dataSource.sectionModels[index].header
            })
    }
    
    // 테이블뷰와 버튼 바인딩
    func bindTableView() {
        let dataSubject = BehaviorSubject(value: initialSections)
        // 테이블뷰 데이터 바인딩
        dataSubject
            .bind(to: tableView.rx.items(dataSource: dataSource))
            .disposed(by: disposeBag)

        // 버튼 클릭 시 데이터 업데이트
        button.rx.tap
            .map { self.updatedSections }
            .bind(to: dataSubject)
            .disposed(by: disposeBag)
	    } 
	}
}
```