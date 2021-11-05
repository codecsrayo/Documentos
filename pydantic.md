Ejemplos de código `@validator()`:
````python
def test_validate_pre():
    @dataclass
    class MyDataclass:
        a: List[int]

        @validator('a', pre=True)
        def check_a1(cls, v):
            v.append('123')
            return v

        @validator('a')
        def check_a2(cls, v):
            v.append(456)
            return v

    assert MyDataclass(a=[1, 2]).a == [1, 2, 123, 456] 
````

[pydantic @validator():](https://www.programcreek.com/python/example/112469/pydantic.validator)
