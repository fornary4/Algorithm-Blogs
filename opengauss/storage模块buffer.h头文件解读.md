## storage模块buffer.h头文件解读

**作用：buffer.h是opengauss的缓冲机制实现**

**缓冲区介绍**

在PostgreSQL中，任何对于表、元组、索引等操作都在缓冲池中进行，缓冲池的数据调度都以磁盘块为单位，需要访问的数据块以磁盘块为单位调用函数smgrread写入缓冲区，而smgrwrite将缓冲池数据写回磁盘。调入缓冲池中的磁盘块称为缓冲区，多个缓冲区组成的缓冲池。

PostgreSQL有两种缓冲池：共享缓冲池和本地缓冲池。共享缓冲池主要作为普通表的操作场所，本地缓冲池则仅本地可见的临时表的操作场所。

**分析**

- 定义了Buffer类来表示缓冲区
- 定义了一系列函数来实现缓冲区的功能
- 实现的功能：初始化缓冲区，清空缓冲区，在缓冲区添加数据，获取缓冲区数据，获取缓冲区中的可用空间



```java
#ifndef MOT_BUFFER_H
#define MOT_BUFFER_H

#define DEFAULT_BUFFER_SIZE 4096 * 1000  // 默认缓冲区大小为4MB

#include <stdint.h>
#include <string>
#include <sstream>
#include "utilities.h"

namespace MOT {
    
//buffer类的定义
class Buffer {
public:
    //Buffer的构造函数
    inline Buffer() : Buffer(DEFAULT_BUFFER_SIZE)
    {}

    inline Buffer(uint8_t* buffer, uint32_t size)
        : m_bufferSize(size), m_nextFree(0), m_buffer(buffer), m_allocated(true)
    {}

    inline Buffer(uint32_t size) : m_bufferSize(size), m_nextFree(0), m_allocated(false)
    {}
	//虚函数，多态的体现，子类可以重写
    inline virtual ~Buffer()
    {
        if (m_buffer) {
            delete[] m_buffer;
        }
    }
	
    //初始化缓冲区
    inline bool Initialize()
    {
        if (!m_allocated) {
            m_buffer = new (std::nothrow) uint8_t[m_bufferSize];//用数组来实现
            if (m_buffer) {
                m_allocated = true;
            }
        }
        return m_allocated;
    }
	//重载运算符
    Buffer& operator=(Buffer&& other) = delete;
    Buffer& operator=(const Buffer& other) = delete;
	
    //清空缓冲区
    inline static std::string Dump(const uint8_t* buffer, uint32_t size)
    {
        std::stringstream ss;
        ss << std::hex;
        for (uint32_t i = 0; i < size; i++) {
            if (buffer[i] < 16)
                ss << "0";
            ss << (uint)buffer[i] << "::";
        }
        return ss.str();
    }
	
    //在缓冲区某处添加数据
    inline bool AppendAt(const void* data, uint32_t size, uint32_t offset)
    {
        if (offset + size <= m_bufferSize) {
            errno_t erc = memcpy_s(&m_buffer[offset], m_bufferSize - offset, data, size);
            securec_check(erc, "\0", "\0");
            return true;
        } else {
            return false;
        }
    }

    //将类型化数据添加到缓冲区
    template <typename T>
    inline bool Append(const T x)
    {
        if (m_nextFree + sizeof(T) <= m_bufferSize) {
            T* ptr = (T*)&m_buffer[m_nextFree];
            *ptr = x;
            m_nextFree += sizeof(T);
            return true;
        } else {
            return false;
        }
    }

    //将原始项添加到缓冲区
    inline bool Append(const void* data, uint32_t size)
    {
        if (m_nextFree + size <= m_bufferSize) {
            errno_t erc = memcpy_s(&m_buffer[m_nextFree], m_bufferSize - m_nextFree, data, size);
            securec_check(erc, "\0", "\0");
            m_nextFree += size;
            return true;
        } else {
            return false;
        }
    }

    //追加输入流中的原始数据
    bool Append(std::istream& is, uint32_t size)
    {
        if (m_nextFree + size <= m_bufferSize) {
            char* ptr = (char*)&m_buffer[m_nextFree];
            is.read(ptr, size);
            m_nextFree += size;
            return true;
        } else {
            return false;
        }
    }

    //获取缓冲区中的数据
    inline void* Data() const
    {
        return m_buffer;
    }

    //检索缓冲区中的数据
    inline void* Data(uint32_t* size) const
    {
        *size = m_nextFree;
        return m_buffer;
    }

    //重置缓冲区
    inline void Reset()
    {
        m_nextFree = 0;
    }

    //获取缓冲区中的最大可用空间。
    inline uint32_t MaxSize() const
    {
        return m_bufferSize;
    }

    ////获取缓冲区中的可用空间。
    inline uint32_t FreeSize() const
    {
        return m_bufferSize - m_nextFree;
    }

    //获取缓冲区的大小
    inline uint32_t Size() const
    {
        return m_nextFree;
    }

    //查询缓冲区是否为空。
    inline bool Empty() const
    {
        return m_nextFree == 0;
    }

    //将缓冲区转存为字符串格式
    std::string Dump() const
    {
        return Buffer::Dump(m_buffer, m_nextFree);
    }

private:
    /** @var Next write offset. */
    uint32_t m_bufferSize;

    /** @var Next write offset. */
    uint32_t m_nextFree;

    /** @var The buffer. */
    uint8_t* m_buffer;

    /** @val indicates whether memory was already allocated for the buffer */
    bool m_allocated;
};
}  // namespace MOT
#endif /* MOT_BUFFER_H */

```

