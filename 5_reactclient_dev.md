# React.js 클라이언트 개발

### 디렉토리 구조
* src
    * api
    * components
    * pages
    * router

### api 받는 곳 세팅
* postApi.js
    ```javascript
    import axios from 'axios';

    export const API_SERVER_HOST = 'http://localhost:8080';
    const prefix = `${API_SERVER_HOST}/api/v1/post`;

    export const getList = async () => {
        const result = await axios.get(`${prefix}`);
        return {
            status: result.status,
            data: result.data
        }
    }

    export const getOne = async (id) => {
        const result = await axios.get(`${prefix}/${id}`);
        return {
            status: result.status,
            data: result.data
        };
    }

    export const postAdd = async (postDto) => {
        const result = await axios.post(`${prefix}`, postDto);
        return {
            status: result.status,
        };
    }

    export const putOne = async (postDto) => {
        const result = await axios.put(`${prefix}/${postDto.id}`, postDto);
        return {
            status: result.status
        };
    }

    export const deleteOne = async (id) => {
        const result = await axios.delete(`${prefix}/${id}`);
        return {
            status: result.status
        };
    }
    ```

### 페이지 제작
* AddPage.js
* ListPage.js
* ModifyPage.js
* ReadPage.js

### 라우터 세팅
* root.js
    ```javascript
    const root = createBrowserRouter(
        [
            {
                path:"",
                element: <ListPage/>
            },
            {
                path:"read/:id",
                element: <ReadPage/>
            },
            {
                path:"add",
                element: <AddPage/>
            },
            {
                path:"modify/:id",
                element: <ModifyPage/>
            }
        ]
    );
    ```

* App.js 에서 router를 기본적으로 쓰도록 수정
    ```javascript
    return (
        <RouterProvider router={root}/>
    );
    ```

### ListComponent 제작
```javascript
import React, {useEffect, useState} from 'react';
import {getList} from "../api/postApi";
import {useNavigate} from "react-router-dom";

const initState = [];

function ListComponent(props) {

    const [serverData, setServerData] = useState(initState);
    const navigate = useNavigate();

    useEffect(() => {
        getList().then(response => {
            console.log(response);
            setServerData(response.data);
        })
    }, []);

    return (
        <div>
            <h3>List Component</h3>
            {serverData.map(post => (
                <div
                    key={post.id}
                    onClick={() => navigate({
                        pathname: `/read/${post.id}`
                    })}
                >
                    <div>{post.title}</div>
                </div>
            ))}
        </div>
    );
}

export default ListComponent;
```

* 어 왜 안돼지?
    > CORS(Cross-Origin Resource Sharing) 정책 때문  
    > Origin : protocol + host + port number  
    > 따라서, 서버측에 CORS 설정을 해줘야 한다  
    > `config/WebConfig.java`
     ```java
    @Configuration
    public class WebConfig implements WebMvcConfigurer {
        @Override
        public void addCorsMappings(CorsRegistry registry) {
            registry
                    .addMapping("/api/**")
                    .allowedOrigins("http://localhost:3000")
                    .allowedMethods("GET", "POST", "PUT", "PATCH", "DELETE")
                    .allowedHeaders("*")
                    .allowCredentials(true);
        }
    }
    ```     
* 어 왜 로그가 두번 찍히지?
    > React.StrictMode 때문.  
    > 리액트가 자식 컴포넌트를 엄격하게 검사함.  
    > 애플리케이션 내 어디서든지 <React.StrictMode> 태그로 감싸주면 strict 모드를 활성화할 수 있다. 그러면 개발모드에서 (개발 단계시 오류를 잘 잡기위해) 두 번씩 렌더링된다.  
    > 따라서, 보기싫으면 `index.js`에서 전체 앱이 React.StrictMode로 감싸져 있는걸 제거하자.

### ReadComponent 제작

* ReadPage
    ```javascript
    const {id} = useParams();
    return (
        <div>
            Read Page
            <ReadComponent id={id}/>
        </div>
    );
    ```
* ReadComponent
    ```javascript
    import React, {useEffect, useState} from 'react';
    import {useNavigate} from "react-router-dom";
    import {getOne} from "../api/postApi";

    const initState = {
        id: null,
        title: null,
        content: null
    };

    function ReadComponent(props) {

        const [serverData, setServerData] = useState(initState);
        const navigate = useNavigate();

        useEffect(() => {
            getOne(props.id).then(response => {
                console.log(response);
                setServerData(response.data);
            });
        }, [props.id]);

        return (
            <div>
                <h3>Read Component</h3>
                <div>제목: {serverData.title}</div>
                <div>내용: {serverData.content}</div>
                <br/>
                <button onClick={() => navigate(`/`)}>리스트로</button>
                <button onClick={() => navigate(`/modify/${serverData.id}`)}>수정</button>
            </div>
        );
    }

    export default ReadComponent;
    ```

### AddComponent 제작
```javascript
import React, {useState} from 'react';
import {postAdd} from "../api/postApi";
import {useNavigate} from "react-router-dom";

const initState = {
    title: '',
    content: ''
};

function AddComponent(props) {

    const [postDto, setPostDto] = useState({...initState});
    const navigate = useNavigate();

    const handleChangePostDto = (e) => {
        console.log(e.target.name, e.target.value);
        postDto[e.target.name] = e.target.value;
        setPostDto({...postDto});
    }

    const handleClickAdd = () => {
        // 프론트엔드 단의 유효성 검사
        if (!postDto.title.trim() || !postDto.content.trim()) {
            alert('제목과 내용을 모두 입력하세요.');
            return;
        }
        postAdd(postDto).then(response => {
            if (response.status !== 201) {
                alert('등록 에러');
                return;
            }
            setPostDto({...initState});
            alert('정상 등록 되었습니다.');
            navigate(`/`);
        }).catch(error => {
            // 백엔드 단의 유효성 검사
            if (error.response.status === 400) {
                alert('제목과 내용을 모두 입력하세요.');
            } else {
                alert('알 수 없는 에러');
            }
        });
    }

    return (
        <div>
            <h3>Add Component</h3>
            제목
            <input
                name="title"
                type="text"
                value={postDto.title}
                onChange={handleChangePostDto}
            >
            </input>
            <br/>
            내용
            <input
                name="content"
                type="text"
                value={postDto.content}
                onChange={handleChangePostDto}
            >
            </input>
            <br/>
            <button
                type="button"
                onClick={handleClickAdd}
            >
                등록
            </button>
            <button onClick={() => navigate(`/`)}>리스트로</button>
        </div>
    );
}

export default AddComponent;
```


### ModifyComponent 제작
```javascript
import React, {useEffect, useState} from 'react';
import {useNavigate} from "react-router-dom";
import {deleteOne, getOne, putOne} from "../api/postApi";

const initState = {
    title: '',
    content: ''
};

function ModifyComponent(props) {

    const [postDto, setPostDto] = useState({...initState});
    const navigate = useNavigate();

    useEffect(() => {
        getOne(props.id).then(response => {
            setPostDto(response.data)
        })
    }, [props.id]);

    const handleChangePostDto = (e) => {
        console.log(e.target.name, e.target.value);
        postDto[e.target.name] = e.target.value;
        setPostDto({...postDto});
    }

    const handleClickModify = () => {
        // 프론트엔드 단의 유효성 검사
        if (!postDto.title.trim() || !postDto.content.trim()) {
            alert('제목과 내용을 모두 입력하세요.');
            return;
        }
        putOne(postDto).then(response => {
            if (response.status !== 200) {
                alert('수정 에러');
                return;
            }
            setPostDto({...initState});
            alert('정상 수정 되었습니다.');
            navigate(`/read/${props.id}`);
        }).catch(error => {
            // 백엔드 단의 유효성 검사
            if (error.response.status === 400) {
                alert('제목과 내용을 모두 입력하세요.');
            } else {
                alert('알 수 없는 에러');
            }
        });
    }

    const handleClickDelete = () => {
        deleteOne(props.id).then(response => {
            if (response.status !== 200) {
                alert('삭제 에러');
                return;
            }
                setPostDto({...initState});
                alert('정상 삭제 되었습니다.');
                navigate(`/`);
        })
    }

    return (
        <div>
            <h3>Modify Component</h3>
            제목
            <input
                name="title"
                type="text"
                value={postDto.title}
                onChange={handleChangePostDto}
            >
            </input>
            <br/>
            내용
            <input
                name="content"
                type="text"
                value={postDto.content}
                onChange={handleChangePostDto}
            >
            </input>
            <br/>
            <button
                type="button"
                onClick={handleClickModify}
            >
                수정
            </button>
            <button onClick={handleClickDelete}>삭제</button>
            <br/>
            <button onClick={() => navigate(`/read/${props.id}`)}>뒤로</button>
        </div>
    );
}

export default ModifyComponent;
```

### Validation
* client validation
* server validation

### 반복되는 부분...
* Custom Hook