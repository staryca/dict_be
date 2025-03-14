## Стварэнне слоўніка для postgresql

### Капіраванне стоп слоўніка
```sh
sudo cp belarusian.stop /usr/share/postgresql/14/tsearch_data/belarusian.stop
```

### Капіраванне граматычных слоўнікаў пры іх адсутнасці
```sh
sudo cp be_by.affix /usr/share/postgresql/14/tsearch_data/be_by.affix
sudo cp be_by.dict /usr/share/postgresql/14/tsearch_data/be_by.dict
```

### Стварэнне слоўніка ў postgresql
```sql
CREATE TEXT SEARCH DICTIONARY belarusian_huns (
    TEMPLATE = ispell,
    DictFile = be_BY,
    AffFile = be_BY,
    StopWords = belarusian
);
```
### Стварэнне слоўніка стоп-слоў
```sql
CREATE TEXT SEARCH DICTIONARY belarusian_stem (
    template = simple,
    stopwords = belarusian
);
```

### Стварэнне канфігурацыі
```sql
CREATE TEXT SEARCH CONFIGURATION belarusian (PARSER=default);
```

### Настройка канфігурацыі
```sql
ALTER TEXT SEARCH CONFIGURATION belarusian ALTER MAPPING 
    FOR hword, hword_part, word WITH belarusian_huns, belarusian_stem;

ALTER TEXT SEARCH CONFIGURATION belarusian ALTER MAPPING 
    FOR int, uint, numhword, numword, hword_numpart, email, float, file, url, url_path, version, host, sfloat WITH simple;

ALTER TEXT SEARCH CONFIGURATION belarusian ALTER MAPPING 
    FOR asciihword, asciiword, hword_asciipart WITH english_stem;
```

### Праверка пошуку
```sql
SELECT _column_,
    ts_rank(to_tsvector('belarusian', _column_), websearch_to_tsquery('belarusian', 'чорны')),
    ts_headline('belarusian', _column_, websearch_to_tsquery('belarusian', 'чорны'))
  FROM _table_ 
  WHERE to_tsvector('belarusian', _column_) @@ websearch_to_tsquery('belarusian', 'чорны')
  ORDER BY ts_rank(to_tsvector('belarusian', _column_), websearch_to_tsquery('belarusian', 'чорны')) DESC;
```
